# Workspace keys

Workspaces are plain text (format 2). The grammar is a header, then `[Channel]`
blocks, each followed by its `[Rtu]` blocks and their point sub-blocks.
Containment is by order; unknown keys are ignored; a block only needs the keys
that differ from the defaults.

```
[Workspace] Format=2

[Channel]
Tool=DNP3
Name=Line 1
Running=1
# endpoint / link / TLS keys ...

  [Rtu]
  Proto=DNP3
  Addr=10
  Name=OS10
  # station keys ...

    [Points]
    # one point per line: group|index|name|...|side|tail
```

Key points:

- **`Format=2`** is required; older formats are not read.
- Masters save **`Running=`**, so a master workspace restarts its channels on
  load.
- Point rows are **side-neutral** — the same row works in a master or a slave
  channel, with `|M` / `|S` marking whose tail follows — so an RTU can be moved
  between a master and a slave.
- Connection keys shared by every tool: `AutoReconnect`, `ReconnectMin`,
  `ReconnectMax`, `RtsMode`, `RtsDelay`, plus the TLS keys. Master channels
  on TCP / UDP also write `LocalInterface` (the IP address of the adapter to
  connect from) when one is set; the P2P Monitor writes `RtuLocalInterface`.

## Tags and scripts

- A tagged point carries two extra fields at the end of its row, `|tagOn|tag`
  (DNP3 / IEC slave points and master map rows); Modbus writes `Cell<i>.TagOn=` /
  `Cell<i>.Tag=` in a master map, `Tag=<i> <on> <name>` in a slave `[Block]` and
  `TagW=` / `TagB=` (word / bit) in a `[PLC]` block.
  Untagged points write nothing, so older files are unchanged.
- An RTU with a script has a `[Script]` block after its points: `On=1` (start
  with the workspace), optional `File=<path relative to the workspace>`, and the
  inline source as lines prefixed with `>` (with `File=` too: the last text read
  from the file, used when the file is missing).
- File paths (`File=`, a display's `Source=`, an Image object's `Text=`) are
  relative to the workspace when the file sits beside it or in its `scripts`,
  `displays` or `images` sub-folder, full paths otherwise.

```
  [Script]
  On=1
  >local cb = Breaker.new{ status = "CB1_ST", travel_ms = 120 }
  >on_command("CB1_CMD", cb:handler())
```

## HMI displays

Displays follow the channels and charts: one `[Display.N]` block per display and
one `[Display.N.Obj.M]` block per object, in drawing order. Only keys that differ
from the defaults are written.

| `[Display.N]` key | |
|---|---|
| `Name`, `Width`, `Height`, `Bg`, `Grid`, `Snap`, `NextId` | the canvas |
| `Scope` | `Channel/RTU` for an RTU's display, `Channel` for a channel's (absent = workspace) |
| `Source` | the `.ssd` file it was imported from, relative to the workspace |
| `Open=0` | the window was closed when saved |

Object blocks start with `Kind=` (`Text`, `Value`, `Led`, `Bar`, `Shape`,
`Breaker`, `Button`, `Setpoint`, `Flow`, `Gauge`, `Image`), `Id=` and `Rect=`
(`x y w h`), then the binding (`Tag`, `FeedbackTag`, `CmdTag`) and appearance
keys (`Shape`, `Radius`, `Dir` for shapes; `VisTag`, `VisOp`, `VisValue` for
visibility). A `.ssd` file (Export, DisplayLab) holds the same blocks. `ShowDisplays=1`
in `[Workspace]` reopens the Displays window.

!!! note "Draft"
    A full per-tool key table will be added here. For now the file itself is the
    best reference — open a saved workspace in a text editor to see every key in
    context.
