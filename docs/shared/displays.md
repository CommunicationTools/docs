# HMI displays

!!! info "SCADAScop 2.0"
    Displays are a SCADAScop feature: they are not available in the standalone tools.

A **display** is an operator screen drawn over the workspace's live points:
values, LEDs, bars, gauges, breakers, power flows, pictures, command buttons and
setpoint fields. A display can show and command points of **master** RTUs (a real
HMI over the devices you poll) and of **slave** RTUs (a front panel for the
simulated device). Displays are saved in the workspace.

## The Displays window

**Tools › Displays** lists the workspace's displays:

| Column | |
|---|---|
| **Show** | tick to open the display's window, untick to close it (clicking the name does not) |
| **Name** | double-click to rename |
| **Scope** | **Workspace**, a *Channel*, or the *Channel/RTU* the display belongs to (see [Scope](#scope-workspace-channel-or-one-rtu)) |
| **File** | the `.ssd` file the display was imported from, or `<local>` |
| **…** | Rename, Move up / down, Export…, Reload from file, Delete (also on right-click) |

**New display** and **Import…** create displays for the scope chosen in the
**for** list next to them. **Reload all** re-reads every display that came from a
file.

A display bound to an RTU or a channel is also listed in its channel's context
menu in the Channels tree (**Displays ›** *RTU: display* / *Channel: display*),
and an RTU's displays on its node in the collapsed Channels rail.

## Binding to points: tags

Objects bind to points by **tag**, the same names [RTU scripts](scripting.md)
use:

- **Slave points** carry the tag you give them in the **Tag** column of their
  Points window.
- **Master points** get one from the **Tag** column of the DNP3 and IEC map
  tables (after Name), or of the Modbus poll window (the **Tag column** checkbox
  next to Display options). Tick the box and type a name; names are unique inside
  the RTU.

A tag's full path is `Channel/RTU/tag`. In the editor, the **Browse** button next
to every tag field opens the tag browser: every tagged point with its channel,
RTU, kind, units and name, filtered by channel, RTU and text — double-click to
bind.

## Scope: workspace, channel or one RTU

- A **Workspace** display binds by full path and can mix points of any channel
  and RTU — an overview of the whole plant.
- A display whose scope is an **RTU** binds that RTU's points by **bare tag name**
  (`CB1_ST` instead of `Feeder 1/OS10/CB1_ST`); a full path still reaches any
  other point. Its title shows the RTU.
- A display whose scope is a **channel** shows several RTUs of one channel: a
  tag is written `{rtu.N}/tag` — the N-th RTU of the channel, in the channel's
  order — or `RTU/tag` by name; a bare tag means the first RTU. The tag browser
  writes `{rtu.N}/tag` for you, so the same file works on another channel whose
  RTUs have other names.

Because a scoped display only names tags, the same drawing works for every RTU
that uses the same tag names: **Export…** it once as a `.ssd` file, then
**Import…** it for each RTU. The file stays the display's source — edit the file
(in SCADAScop or [DisplayLab](#displaylab)), then **Reload all**, and every RTU
that uses it is updated. An RTU can have several displays.

In any text these placeholders are replaced, so one file shows each RTU's own
names:

| Placeholder | Replaced by |
|---|---|
| `{rtu}` | the display's RTU; in a channel display, the channel's first RTU |
| `{rtu.N}` | the N-th RTU of the display's channel (`{rtu.1}` is the first); `?` when the channel has fewer |
| `{channel}` | the display's channel |
| `{scope}`, `{display}` | the scope as shown in the list, the display's name |

## Editing

The toolbar switches between **Run** and **Edit**. In edit mode:

- **Add..** lists the objects; click on the canvas to place one.
- Click to select; drag to move, drag a handle to resize; arrows nudge by one
  pixel (<kbd>Shift</kbd>: one grid step).
- <kbd>Ctrl</kbd>+click adds or removes objects from the selection,
  <kbd>Ctrl</kbd>+<kbd>A</kbd> selects all, dragging on an empty area selects
  everything inside the rectangle. A selection moves, duplicates and deletes as
  a group.
- With several objects selected the properties panel (and the right-click menu)
  aligns them to the **primary** one — the one with the handles: left, center,
  right, top, middle, bottom, same width / height / size — and **Distribute H /
  V** spaces three or more evenly.
- **Undo** / **Redo** (<kbd>Ctrl</kbd>+<kbd>Z</kbd> / <kbd>Ctrl</kbd>+<kbd>Y</kbd>),
  **Duplicate** (<kbd>Ctrl</kbd>+<kbd>D</kbd>), **Raise** / **Lower**, **Delete**
  (<kbd>Del</kbd>), **Grid**, **Snap** and the zoom.
- The properties panel edits the display (name, size, grid, background) and the
  selected object.

## Objects

| Object | Shows | Notes |
|---|---|---|
| **Text label** | a fixed caption | font, size, bold / italic, alignment, `{rtu}`-style fields |
| **Value readout** | the tag's value | prefix, decimals, units; a bad quality greys and strikes it |
| **LED / indicator** | a colour per state | states with label, colour and blink; numbers pick a state by thresholds |
| **Bar / tank** | a fill between Min and Max | vertical or horizontal, optional segments (a battery) |
| **Gauge** | an arc between Min and Max | readout and needle can be hidden; arc width |
| **Power flow** | moving dashes along a line | Min..Max: the middle stops it, above flows one way, below the other — set Min = −Max for a signed power |
| **Breaker / switch** | open / closed / intermediate / indeterminate | square, circle or knife symbol, vertical or horizontal, mirrored blade, label position; a **command tag** makes it operable |
| **Command button** | a caption and the state of its feedback tag | action: two-state (Open / Close), raise / lower, pulse, set value; state text can be hidden |
| **Setpoint entry** | the value; click to type a new one | with a **feedback tag** it shows the value the RTU reports back |
| **Shape** | rectangle, rounded rectangle, ellipse, triangle (pointing up / right / down / left), diamond, line | optional tag: **colour by state** (see below); **Add.. › Line** draws an exact line by **Length** and **Angle** (0 horizontal, 90 vertical, 45 rising, 135 falling) |
| **Image** | a PNG or SVG file | transparency kept, keep-aspect, tint; keep pictures beside the workspace or in an `images` folder so it stays portable ([Files the workspace uses](workspaces.md#files-the-workspace-uses-scadascop)) — lower it under the live objects and use it as the single-line diagram |

A binary point on a breaker reads 0 as open and 1 as closed; a double-bit point
uses its four codes. Text objects can use the **Sans** or **Mono** face (from the
system's fonts — Segoe UI / Consolas on Windows, DejaVu or Liberation on Linux)
with **bold** and **italic**; where a face is missing the UI font is used.

**Colour by state.** Give a shape a tag and press **Add colour states**: the
shape takes the colour of the state its value selects — Off / On for a binary
point, the double-bit codes, or thresholds for a number (add states with
**+ state**) — and can blink. A line changes its stroke. Remove every state to
go back to fixed colours.

**Visibility.** Every object has a **Visibility** section: *Always visible*, or
visible while a tag's value is `=`, `!=`, `>`, `<`, `>=` or `<=` a number (a
binary point is 0 / 1; a double-bit 0 intermediate, 1 open, 2 closed, 3
indeterminate). While the tag has no value the object is hidden. A hidden
object cannot be clicked; in edit mode it stays visible, dimmed and hatched, so
you can still edit it.

In run mode, hovering an object shows its tag, value, state and quality (turn
**Hint on hover** off per object). An object bound to a tag that has no value is
outlined in magenta.

## Commands

Buttons, breakers with a command tag and setpoint fields send commands. With
**Two-step** on (the default) a command follows select-before-operate, as in the
point windows:

1. Click the object: a dialog shows the state buttons (*Open* / *Close*, *Raise* /
   *Lower*, …) and the current feedback.
2. A state button sends the **SELECT**; the dialog waits for the RTU's
   confirmation.
3. Once selected, **Execute** sends the operate and **Cancel** abandons it. The
   dialog reports the outcome (done, refused, timed out).

With Two-step off, the first click executes directly.

What goes out depends on whose point it is:

- **A master point** — the master sends the real protocol command: DNP3 CROB
  (trip / close pulse, or latch) or analog output, IEC C_SC / C_DC / C_RC / C_SE,
  Modbus FC05 / FC06 / FC16. The master's own script and logs see it like a
  command from its point window.
- **A slave's own point** — no protocol is involved: the display drives the
  simulation, writing the command point and moving its feedback point at once.

## DisplayLab

**DisplayLab** is a separate development tool (built with *BuildAll*, not part of
the release) for drawing and tuning displays without a plant: it simulates tags
(hold, sine, ramp, random walk, toggle, pulse) and answers commands with
configurable select / execute delays, refusals and feedback. Its `.ssd` files are
the same as SCADAScop's exports.

## Workspace storage

Displays are saved in the `.ssw` after the charts, as `[Display.N]` blocks with
one `[Display.N.Obj.M]` block per object; see
[Workspace keys](../reference/workspace-keys.md#hmi-displays).
