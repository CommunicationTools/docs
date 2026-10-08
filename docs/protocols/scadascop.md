# SCADAScop

!!! note "Draft — expanding"
    Outline chapter.

SCADAScop embeds every Master and Slave in one shell, organised into **groups**,
with a single **Communication Monitor** across all protocols.

## Highlights

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
