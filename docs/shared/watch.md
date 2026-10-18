# Watch window

**Tools › Watch** (SCADAScop) keeps an eye on a hand-picked set of points while
you test: their live value, quality, when they were last updated and how often
they changed. It is read only — nothing in the window writes a value or sends a
command, so it is safe to leave open beside a test run.

## Adding points

Points of any protocol and role can sit in the same list:

- **Add points…** in the window's toolbar lists every point by channel › RTU ›
  type. Filter by name, tag, type or key; click / Ctrl+click / Shift+click to
  select, double-click to add one, **Add N to …** for the selection.
- **Right-click a row** of a master's map (DNP3, IEC 104 / 101) or poll window
  (Modbus) › **Add to watch**. In a Modbus poll window the row header's menu adds
  the whole row.
- **Right-click a point** in a slave's points window (DNP3, IEC 104 / 101) or a
  cell of a Modbus slave's block / PLC memory › **Add to watch**.
- **Drag** rows from a master's map / poll window onto the Watch window, exactly
  as onto a chart.

Every point of every master and slave RTU can be watched, tagged or not. The
picker lists a Modbus PLC-style slave's words that have a name, a display format
or a tag (its 64K memory is not listed cell by cell) — any other address is added
from the memory window's right-click menu.

!!! note "Control and setpoint points of a master"
    A master's command point shows the last value the RTU **accepted**: a Modbus
    write map's cell the value of its last successful write (FC05 / 06 / 15 / 16),
    an IEC 104 / 101 control the value confirmed by the positive ACTCON of its
    execute (also shown in the map's Value column). Until a command is sent the
    row shows *no value*.

!!! note "Slave points"
    A slave point is read straight from the simulated RTU, so on a slave row
    **Updated** and **Age** tell when the value (or its quality) last *changed* —
    by the simulation, a script, a display or a master's command.

## Reading the table

| Column | |
|---|---|
| **Name** | the point's name, its tag, or a label of your own (double-click) |
| **Value** | engineering value with units; binaries as `ON (1)` / `OFF (0)`, double-bits as `OFF (1)`, `ON (2)`, `INTERMEDIATE (0)`, `INDETERMINATE (3)`. Tinted orange when the quality is not good |
| **Quality** | decoded flags: DNP3 `ONLINE`, `COMM_LOST`, `RESTART`, `LOCAL_FORCED`, `OVER_RANGE`…; IEC `good`, `IV`, `NT`, `SB`, `BL`, `OV`. Modbus carries none (`-`); hover for the raw byte |
| **Updated** / **Age** | arrival time of the last update (in the Time & Date frame) and how long ago that was — a stale point stands out by its age |
| **Changes** | value changes since the point was added or the counter reset; hover for the time of the last change |
| **Protocol**, **Channel**, **RTU**, **Type**, **Address** | where the point lives |
| **Time tag**, **Key** | hidden by default: the RTU's own time stamp and the point key |

A row flashes (Animation Colors › updated) when its value changes. Right-click
the header to show, hide or reorder columns.

## Lists and rows

- Several lists as tabs: **+** adds one, double-click a tab (or right-click ›
  **Rename…**) to rename it, close it or right-click › **Delete list** to remove it.
- Select rows with click / Ctrl / Shift; **Delete** removes them, **Ctrl+C** copies
  them as TAB-separated text (pastes into a spreadsheet).
- Right-click a row: **Label…**, **Move up / down**, **Copy key**, **Add to chart**,
  **Copy to** another list, **Reset counter**, **Remove**.
- Drag a row onto a chart to plot it.
- **Freeze** holds the values shown while the channels keep running (Changes
  keeps counting); untick it to catch up at once. **Reset counters** zeroes the
  Changes column of the list.

The lists are saved in the workspace (`[Watch.N]` blocks, see
[Workspace keys](../reference/workspace-keys.md)).
