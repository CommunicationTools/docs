# Release notes

Each tool is versioned independently and published on its GitHub release page
under [github.com/CommunicationTools](https://github.com/CommunicationTools). Use
**Help → Check for updates…** in a tool to see whether a newer release exists
(see [Updates](shared/updates.md)).

## Current versions

| Tool | Version |
|---|---|
| ModbusScop Master | 2.7 |
| ModbusScop Slave | 2.7 |
| DNPScop Master | 1.8 |
| DNPScop Slave | 2.7 |
| IEC104Scop Master / Slave | 1.8 / 1.7 |
| IEC101Scop Master / Slave | 1.8 / 1.9 |
| SCADAScop | 1.9 |

Each tool is published in three builds — **Windows 10 / 11** (`win64`),
**Windows 7 SP1** (`win7`) and **Linux x86_64** (`linux-x86_64`); see
[Install](getting-started/install.md) and [Running on Linux](getting-started/linux.md).

!!! note "The standalone tools are now frozen"
    This is the last feature release of the standalone tools. New features are
    developed in **SCADAScop**, which contains every tool; the standalone tools
    remain available and receive fixes.

## Highlights of this release

- **All tools** — settings, window layout, recent-files lists and the shared
  colours file now live in your user folder (`%APPDATA%\CommunicationTools`,
  Linux `~/.config/CommunicationTools`) instead of next to the executable, so a
  tool can run from a read-only folder. Existing files beside the executable are
  copied over on first start; `SCOP_USER_DIR` picks another folder. See
  [Install](getting-started/install.md).
- **All master tools, SCADAScop, P2P Monitor** — a **Local interface** field on
  TCP / UDP channels: the IP address of the network adapter the connection must
  leave from, for PCs with several adapters in the same network (empty = the
  system chooses, as before). See [Channels](shared/channels.md).
- **All tools** — released for **Windows 7** and **Linux** besides Windows 10 /
  11. **Help → About** shows which build you are running, and
  **Check for updates** only offers releases that have a file for your build.
  See [Updates](shared/updates.md) and [Running on Linux](getting-started/linux.md).
- **All tools** — the refresh-rate cap now limits drawing only: polling,
  simulation, logging and the web interface keep their rhythm whatever the cap,
  and an unchanged picture is not redrawn — CPU (software) rendering is much
  lighter.
- **ModbusScop Slave** — **RTU Settings → On write** chooses where an accepted
  FC06 / FC16 write is stored: the holding map (default), the input map (read it
  back with FC04) or nowhere.
- **IEC104Scop / IEC101Scop Slave** — a normalized / scaled measurand outside
  its representable range goes out with the **OV** quality bit set on the wire
  only; the point itself is unchanged.
- **SCADAScop** — switching the Channels panel between **Free** and **Docked** no
  longer flickers.

## Previous release

- **DNPScop Master / Slave, SCADAScop** — the Communication Monitor's **Decode**
  column no longer shows a wrong message type (for example
  `DISABLE_UNSOLICITED`) on long application messages sent in several packets.
  The first packet shows its function with *(first seg, more follow)*; the
  following ones are shown as *fragment cont. seg N* / *fragment final seg N*.
  See [Communication Monitor](shared/monitor.md#long-dnp3-messages).
- **IEC101Scop / IEC104Scop Master** — in **Select then execute** command mode
  the master now waits for the positive confirmation (ACTCON) of the SELECT
  before it sends the EXECUTE. A denied selection is not executed, and a
  timeout cancels the pending execute. See
  [Send a control](how-to/send-a-control.md).
- **IEC101Scop Slave** — with **Single-character ACK (E5)** enabled, a class
  poll with nothing to report is answered with the single character `0xE5`
  instead of a fixed-length "no data" frame, and the *NACK class 2*
  communication test answers with `0xA2`.
- **SCADAScop** — new **Tools → Floating-Point Converter**, an IEEE 754
  calculator for FP16 / FP32 / FP64 with Modbus register byte orders. See
  [SCADAScop](protocols/scadascop.md).

The full change history for each release is on its GitHub release page.
