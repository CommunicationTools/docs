# Modbus

!!! note "Draft — expanding"
    This chapter is an outline. The shared windows it refers to are complete in
    [Using the tools](../shared/channels.md); DNP3 is the fully worked example.

ModbusScop is the Modbus pair: **ModbusScop Master** (client) and **ModbusScop
Slave** (server), over Modbus RTU (serial) or Modbus TCP. ModbusScop Slave can
also act as a **gateway** (MBAP ⇄ RTU).

## Addressing

A channel carries one or more **devices**, each identified by a **Unit ID**. On
serial / RTU framing, unit 0 is broadcast and 248–255 are reserved; on MBAP,
0 / 255 mean "the device itself". The tools show a note under the unit-id field.

## Points / maps

Devices expose the four Modbus tables — **Coils**, **Discrete Inputs**,
**Holding Registers**, **Input Registers** — plus a free PLC memory map in the
Slave. The register grids are resizable and have a header right-click menu
(rows per column, hide Name, show address in cell, Base 1 register numbers,
reset widths); see [Points and maps](../shared/points.md).

## Master

- Each RTU opens in **one window with a tab per poll map** (SCADAScop 2.0); each
  map reads / writes a function code over an address range, with the data grid
  showing values live and grouping 32/64-bit values.
- In SCADAScop the **Tag column** checkbox (next to Display options) names cells
  for [HMI displays](../shared/displays.md); a tagged coil map (FC05 / FC15) or
  register map (FC06 / FC16) is commandable from a display.
- Inspect / device-scan / register-scan tools help probe an unknown device.

## Slave

- Serve values from the device tables or the PLC map; edit or simulate them.
- **RTU Settings › On write** chooses where an accepted FC06 / FC16 write lands
  (holding map, input map, or nowhere) and **On coil write** the same for FC05 /
  FC15 (coil map, discrete-input map, or nowhere).
- The PLC memory window has a **Words** / **Bits** view; tag single bits for a
  [script](../shared/scripting.md) from the Bits view.
- **Gateway** mode re-frames MBAP ⇄ RTU with unit remapping and the right Modbus
  exceptions (0x0A / 0x0B) when the device side is down.

## Value lists — enumerations and bit names

Both tools let an RTU carry named **value lists**, so a cell shows what a number
*means* and a write can be chosen by name:

| Kind | Rows | Example cell |
|---|---|---|
| **Enumeration** | value → name (`0 Open`, `1 Closed`, `0x0F Fault`) | `Closed (1)` — a value that is not in the list shows as `7 (?)` in the warning colour |
| **Bit names** | bit → name (`0 RUN`, `1 ALARM`, `5 TRIP`) | `RUN \| TRIP` — nothing set: `(none)`; a set bit without a name: `b7` |

A list is an overlay on the cell's display format: the cell keeps its format and
byte order (U16, Hex, U32 … — floats take no list), the raw value stays in the
text, the hover tooltip shows the whole table and the format badge gets an
**E** / **B** marker. Coils and discrete inputs take a two-row enumeration that
names their OFF / ON states.

**Creating and editing.** Every Modbus RTU has a **Value Lists** entry at the end of
its children in the Channels tree (right-click: New enumeration, New bit names,
Paste list, Import list) and a trailing **Lists** tab in its window (the Master's
Maps window, the Slave's Points window). The tab lists the RTU's lists on the
left and edits the selected one on the right: name, kind, the rows (an
enumeration's Value / Name table with an add line; a bit-names table of 16 / 32 /
64 rows), "Used by" (the cells referencing it), **Copy rows** / **Paste rows** —
paste a `value name` table copied from a device manual or a spreadsheet (tab,
comma, `=` or spaces between value and name) and it becomes the list.

**Assigning.** Right-click a cell → **Enumeration ›** or **Bit names ›** and pick
one of the RTU's lists, `(none)`, or **New list…** (creates one of that kind,
assigns it and opens the Lists tab).

**Writing.** The names follow the value everywhere it can be written:

- Master, double-click a read cell: the write command shows a drop-down of
  `name (value)` for an enumeration (with a *Custom value…* entry for a value
  outside the list), checkboxes per named bit for bit names, and the OFF / ON
  buttons of a coil carry its names.
- Master write maps (FC06 / FC16) and Slave points / PLC memory: an enumeration
  cell edits inline as a drop-down; a bit-names cell shows the set names and
  opens the bit editor on double-click. The Slave's command maps (FC05 / 06 / 15 / 16)
  show the names of what the master wrote.
- Both bit editors gain a **Bit names** section (the same value by name), or show
  the enumeration's name for the current value.

**Moving lists around.** Lists belong to the RTU, so Duplicate / Copy / Export RTU
carry them, and a list travels between the Master and the Slave with the RTU.
A single list moves with **Copy** / **Paste** (clipboard) or **Export…** /
**Import…** (`.mbvl` file) from the tree or the Lists tab. In the workspace they
are `[ValueList]` blocks under the `[Rtu]`; a cell refers to one by name
(`Cell<i>.List=` in a master `[Map]`, `List=<i> <name>` in a slave `[Block]` /
`[PLC]`) — see [Workspace keys](../reference/workspace-keys.md).

## To document

- Poll window layout and the function-code picker.
- Write single / multiple, and the confirmation flow.
- Simulation modes and the Test (no-reply / delay / forced exception) controls.
- Gateway setup end to end.
