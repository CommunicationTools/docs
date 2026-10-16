# SCADAScop

!!! note "Draft — expanding"
    Outline chapter.

SCADAScop embeds every Master and Slave in one shell, organised into **groups**,
with a single **Communication Monitor** across all protocols.

## Highlights

- **[RTU scripting](../shared/scripting.md)** (2.0) — every slave RTU can run a
  Lua script that accepts, refuses or delays commands and drives the simulation.
- **[HMI displays](../shared/displays.md)** (2.0) — **Tools › Displays**: operator
  screens over any master or slave point, with select-before-operate commands.
- **[Windows and monitors](../shared/windows.md)** (2.0) — drag any window out of
  the main window onto a second monitor, dock windows together there, or open a
  **Desktop** (View › Desktops) to dock into.
- **[scopctl](../shared/scopctl.md)** — a terminal client for status, start /
  stop, simulation, scripts and automated tests, across several instances.

- One window hosting all the protocol tools; channels are organised into groups.
- **View → Communication Monitor** shows every embedded tool's traffic in one
  table (with a Tool column and per-tool filters), and a mixed-protocol PCAP that
  Wireshark dissects correctly. **New monitor** opens more instances.
- **Start group / Stop group**, <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>S</kbd> /
  <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>X</kbd>, and a prompt before stopping
  several running channels.
- One self-contained `.ssw` workspace holds every embedded tool's channels.
- Export group / Import channels here on a group's menu.
- **Tools → Floating-Point Converter** is an IEEE 754 calculator for FP16, FP32
  and FP64: type a decimal, paste raw Modbus register contents as hex, or click
  individual bits, and the other views follow. Pick the byte / word order
  (ABCD, CDAB, BADC, DCBA and the one- and four-register equivalents) to read
  registers the way the device sends them. It shows the exact value stored,
  the conversion error and the sign / exponent / mantissa breakdown.

## To document

- The group model and layout.
- The unified monitor in depth.
- Per-tool differences when embedded vs standalone.
