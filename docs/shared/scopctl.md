# scopctl — terminal client

**scopctl** controls and watches one or several running SCADAScop instances —
with a window or [headless](headless.md), on this computer or over the network —
from a terminal or a test script. It talks to SCADAScop's **web interface**, so
start that first (**Tools › Web Interface…**, or `--web <port>` headless).

```
SCADAScop --headless plant.ssw --start-all --web 8080
scopctl                       full-screen terminal interface (":" opens a command line)
scopctl status --rtus
```

It needs a terminal with VT support: Windows 10 or later, or any Linux terminal.

## Connecting

By default scopctl connects to `http://127.0.0.1:8080/`. Give another address
with `-u http://host:port/?token=…`, or set `SCOPCTL_URL` (and `SCOPCTL_TOKEN`).

## Commands

| Command | |
|---|---|
| `scopctl` | the full-screen interface: **F1** overview, **F2** channels, **F3** Status Messages, **:** a command line |
| `scopctl shell` | a prompt with history and Tab completion of commands and names |
| `status [--rtus] [--json]` | groups, channels and their state (`--rtus` adds RTUs and their scripts) |
| `start` / `stop all \| group <name> \| channel <name>` | start or stop channels |
| `sim`, `sim on [speed]`, `sim off`, `sim speed <0.1..10>` | the slave [simulation](simulation.md) |
| `script [status [<channel>[/<rtu>]]]` | the slave RTUs' scripts and their state |
| `script start \| stop \| reload <channel>/<rtu>` | control one RTU's [script](scripting.md) |
| `log [-f] [-n <lines>]` | Status Messages (`-f` follows) |
| `wait <condition> [--timeout <s>]` | wait until a condition holds (default 30 s) |
| `assert <condition>` | fail unless it holds now |
| `list instances \| groups \| channels \| rtus` | names, for scripts and completion |
| `run <file> [--keep-going] [--quiet] [--json]` | run a script of these commands |
| `version` | the scopctl version |

Names with spaces go in quotes: `start channel "Feeder 1"`. An RTU is given by
name or address: `"Feeder 1/OS10"`, `"Feeder 1/10"`.

## Scripts on the RTUs

```
scopctl script status
scopctl script reload "Feeder 1/OS10"
scopctl status --rtus
```

`script status` lists every slave RTU with a script — running or not, its file,
and how many handlers, timers and waiting coroutines it has. `start` runs the
saved source (or the bound file), `reload` re-reads the file first, `stop` stops
it. These are the same controls as the RTU's Script window and the web page.

## Conditions

For `wait` and `assert`:

- `channel <name> connected|waiting|error|stopped|running`
- `group <name> connected|running|stopped`, `all connected|running|stopped`
- `rtu <channel>/<rtu> online|offline`, `sim on|off`
- `channels running|connected|waiting|error|stopped <op> <n>`,
  `rtus online|offline <op> <n>`, `errors|timeouts <op> <n>` with `<op>` one of
  `== != < <= > >=`

## Test scripts

A `.scop` file holds one command per line, `#` for comments, and
`source <file>` to include another. The run stops at the first failure
(`--keep-going` runs on) and its exit code is the first failure's:

```
# smoke.scop
start all
wait all connected --timeout 20
assert errors == 0
script reload "Feeder 1/OS10"
sleep 5
assert rtus offline == 0
stop all
```

Exit codes: 0 done, 1 bad command line, 2 cannot reach SCADAScop, 3 token
refused or monitoring-only, 4 no such group / channel / instance, 5 unexpected
answer, 6 `wait` timed out, 7 `assert` failed.

## Several instances (fleet)

Repeat `-u`, set `SCOPCTL_URL="a;b"`, or list them in a fleet file and pass
`-F plant.fleet`:

```
# plant.fleet
exe = C:\Scop\SCADAScop.exe
north = 8080 run north.ssw start-all log north.log
south = 8081 run south.ssw
far   = http://10.0.0.5:8080/?token=abcd        # attach only
```

`@north` (or `@all`) in a command addresses some instances; a name can carry its
instance (`start channel south:RTU-7`). `scopctl -F plant.fleet up` starts the
`run` instances headless on this computer and waits until they answer; `down`
ends them; `shutdown` ends any headless instance addressed (a SCADAScop with a
window refuses). Each instance is named after SCADAScop's instance name
(`--name`, or the workspace's **File › Instance Name**).

!!! warning "Tokens travel in clear"
    The web interface is plain HTTP. Keep fleet files, which hold tokens, out of
    version control and prefer the loopback address.
