# Communication Tools

A suite of Windows desktop tools for testing, simulating and monitoring SCADA
telecontrol protocols. Each protocol ships as a **Master** (front-end /
control-centre side) and a **Slave** (RTU / outstation side), and **SCADAScop**
embeds all of them in one shell with a unified Communication Monitor.

Created and developed by **Carlos Nardi**.

| Protocol | Master | Slave |
|---|---|---|
| Modbus (RTU / TCP) | ModbusScop Master | ModbusScop Slave |
| DNP3 (IEEE 1815) | DNPScop Master | DNPScop Slave |
| IEC 60870-5-104 | IEC104Scop Master | IEC104Scop Slave |
| IEC 60870-5-101 | IEC101Scop Master | IEC101Scop Slave |
| All of the above | **SCADAScop** (embeds every tool) | |

## What you can do with them

- **Poll and command real equipment** from the Master tools, or **simulate an
  RTU** with the Slave tools, over serial or TCP (with optional TLS).
- **Watch the wire** in the Communication Monitor — every frame, decoded layer
  by layer — and capture it to a **PCAP** that Wireshark can open.
- **Exercise a device's error handling** with the fault-injection panels
  (deny a poll, corrupt a checksum, send a negative confirmation, ...).
- **Build large test sets** fast: create up to 64 channels at once, duplicate
  channels and RTUs, and save everything in one workspace file.

## How this manual is organised

- **[Getting started](getting-started/install.md)** — install, first run, and
  the handful of concepts (channel, RTU/station, point) that every tool shares.
- **[Using the tools](shared/channels.md)** — the windows and features common to
  every tool: Channels, Points, the Monitor, PCAP, workspaces, TLS, themes.
- **[Protocols](protocols/dnp3.md)** — one chapter per protocol, with its
  Master- and Slave-specific windows and worked examples.
- **[How-to](how-to/poll-a-device.md)** — short, task-focused walk-throughs.
- **[Reference](reference/shortcuts.md)** — shortcuts, workspace keys and
  message catalogs.

!!! note "Work in progress"
    This manual is being written alongside the tools. Pages marked *Draft* are
    outlines that are still being filled in. The shared chapters and the DNP3
    chapter are the most complete; use them as the model for the rest.
