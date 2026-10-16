# RTU scripting

!!! info "SCADAScop 2.0"
    Scripting is a SCADAScop feature: it is not available in the standalone tools.

A **slave RTU** — DNP3 outstation, IEC 104 / IEC 101 station or Modbus device —
can run a **Lua script** that makes it behave like a real device in the field:
accept or refuse commands with the protocol's own answer, answer late, move a
breaker through its travel time, step a tap changer, compute a bay's currents
and powers, update counters. Without a script the RTU behaves exactly as before;
a script can be added to an existing configuration point by point.

Each RTU has its own script and its own Lua state. A script cannot touch files or
the network (no `io`, `os`, `require` or `load`), and a runaway handler is
stopped without stopping the script or the protocol.

## 1. Tag the points the script uses

A script never sees groups, indexes or addresses — only **tags**. Open the RTU's
points (right-click the RTU › **Points…**) and use the **Tag** column:

- tick the box of a point and type a name: letters, digits and `_`, not starting
  with a digit, not a Lua keyword (`CB1_CMD`, `P_MW`, `tap_pos`);
- names are unique inside the RTU; a red field is an invalid or duplicated name.

In the Modbus slave the column is shown by the **Tag column** checkbox next to
**Display options** (both the block grids and the PLC memory; in the PLC memory a
bit is tagged from the **Bits** view). A tag cannot point inside a 32 / 64-bit
value — tag its first register.

Tags are also what [HMI displays](displays.md) bind to, so the same names serve
both.

## 2. Open the Script window

Right-click the RTU › **Script…**. The window has:

- a toolbar — **Run** / **Stop**, **Reload** (re-runs the script with the editor's
  changes), the state (stopped, running, error) and **Library** (the standard
  library, read-only);
- the **editor** — Lua colouring, line numbers, the last error marked on its line
  (click the error in the console bar to jump to it), line / column readout;
- a **console** with `print` output and errors (errors also go to Status
  Messages, prefixed `Script:`);
- the RTU's **tags**, each with marks for the handlers that use it
  (*cmd*, *sp*, *chg*); click one to copy its name.

**Bind file…** keeps the source in a `.lua` file instead of the workspace: the
editor turns read-only and the script reloads whenever the file changes, so you
can edit it in another editor. Put the file next to the workspace or in a
`scripts` folder beside it and the workspace stays portable; the workspace also
keeps a copy, which runs when the file is missing (see
[Files the workspace uses](workspaces.md#files-the-workspace-uses-scadascop)). **Import…** / **Export…** copy a `.lua` file in or
out. A script that is running when the workspace is saved starts again when it is
opened.

Type `help()` in a script (or read **Library**) for the whole API.

## 3. Write handlers

```lua
-- A breaker with 120 ms travel, its status on a double-bit point
local cb = Breaker.new{ status = "CB1_ST", travel_ms = 120 }
on_command("CB1_CMD", cb:handler())

-- A setpoint that refuses values out of range
on_setpoint("P_SET", function(sp)
    if sp.value < 0 or sp.value > 50 then return REJECT("NOT_SUPPORTED") end
    set("P_MW", sp.value)
end)

-- Every second: publish a measurand
every_ms(1000, function()
    set("F_HZ", 60 + random() * 0.02 - 0.01)
end)
```

**Decisions.** A command or setpoint handler returns `ACCEPT` (also when it
returns nothing) or `REJECT(code)`. The code becomes the protocol's refusal:

| Protocol | `REJECT` becomes |
|---|---|
| DNP3 | the CROB / analog output status code (`NOT_SUPPORTED`, `HARDWARE_ERROR`, `LOCAL`, …) |
| IEC 104 / 101 | a negative ACTCON (the code goes to the trace) |
| Modbus | an exception: `ILLEGAL_VALUE` → 03, `HARDWARE_ERROR` → 04, `BUSY` → 06, or a number |

**Late answers.** A second return value delays the protocol answer in
milliseconds — a slow RTU, or a test of the master's timeout:

```lua
on_command("CB1_CMD", function(cmd) return ACCEPT, 1500 end)
on_command("CB2_CMD", function(cmd) return REJECT("HARDWARE_ERROR"), 300 end)
```

**Waiting.** Handlers and timers run as coroutines: `wait_ms(ms)` suspends one
and resumes it later, so a handler can model a sequence. A handler that waits
has accepted the command; the rest runs on later ticks.

**Selections.** A select-before-operate command calls the handler twice — once
with `cmd.phase == "SELECT"` (refuse it to deny the selection) and once for the
operate / execute.

**After `ACCEPT`** the tool's built-in behaviour still runs (output status
feedback, the point's links), so a handler only has to add what is special.
`REJECT` runs nothing.

## 4. React to changes and events

```lua
on_change("CB1_ST", function(value, old) print("breaker", old, "->", value) end)
on_event("master_online",  function(info) print("master connected:", info.peer) end)
on_event("sim_on",         function() print("simulation started") end)
```

`on_change` fires for any writer — the master, the simulation, the Points
window, another handler (and the script's own writes: guard against loops).

## 5. Use the standard library

| Object | What it models |
|---|---|
| `Breaker.new{ status, travel_ms, stuck }` | open / close with an intermediate state for the travel time; `:handler()` is a ready-made `on_command` handler; `stuck = true` refuses with `HARDWARE_ERROR` |
| `Bay.new{ Un_kV, Pmax_MW, pf, f_Hz, load, … }` | a feeder bay: `:step(dt)` then read `P_MW`, `Q_MVAr`, phase currents and voltages, `f_Hz`, `E_kWh` |
| `TapChanger.new{ min, max, pos, delay_ms, position }` | `:raise()`, `:lower()`, `:move_to(n)` one step per `delay_ms`, `:ratio()` |
| `Signal.ramp` / `.sine` / `.walk` / `.lag` | value generators: `:step(dt)` returns the next value |
| `LoadProfile.hourly{ [0]=…, … [23]=… }` | a daily load curve: `:at(now_ms())` |

Every function is in the [Script API reference](../reference/script-api.md). A
complete example — breaker, bay, tap changer and energy counter on a DNP3
outstation — ships as `docs/sample-workspaces/bay_demo.lua`.

## Running and controlling scripts

- Scripts tick with the channel (every 20 ms), not with the screen refresh, and
  keep running headless.
- A runtime error is reported and the handler stays registered (the next event
  tries again); a script that exceeds the instruction limit, or does not load, is
  stopped.
- The [web interface](headless.md) shows each RTU's script state and starts,
  stops or reloads it; so does [scopctl](scopctl.md#scripts-on-the-rtus):
  `scopctl script status`, `scopctl script reload "Feeder 1/OS10"`.

## Protocol details in `cmd` and `sp`

| | DNP3 | IEC 104 / 101 | Modbus |
|---|---|---|---|
| command | CROB (group 12) | C_SC, C_DC, C_RC | FC05 / FC15 on a tagged coil (or a command map) |
| `cmd.state` | — (`cmd.tcc` = `TRIP` / `CLOSE`, `cmd.op` = `LATCH_ON` …) | `ON` / `OFF`, `HIGHER` / `LOWER` | `ON` / `OFF` |
| `cmd.phase` | `SELECT`, `OPERATE`, `DIRECT`, `DIRECT_NOACK` | `SELECT`, `EXECUTE` | — |
| setpoint | analog output (group 41) | C_SE_NA / NB / NC, C_BO | FC06 / 16 / 22 / 23 on a tagged register, decoded with the cell's display format |
| `REJECT` | status code | negative ACTCON | exception, the whole multi-write dropped |

In Modbus the handler runs before the value is stored (`get()` still returns the
old value; `sp.value` is the new one). Modbus has no quality flags.
