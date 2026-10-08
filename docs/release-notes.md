# Release notes

Each tool is versioned independently and published on its GitHub release page
under [github.com/CommunicationTools](https://github.com/CommunicationTools). Use
**Help → Check for updates…** in a tool to see whether a newer release exists
(see [Updates](shared/updates.md)).

## Current versions

| Tool | Version |
|---|---|
| ModbusScop Master | 2.6 |
| ModbusScop Slave | 2.6 |
| DNPScop Master | 1.7 |
| DNPScop Slave | 2.6 |
| IEC104Scop Master / Slave | 1.7 / 1.6 |
| IEC101Scop Master / Slave | 1.7 / 1.8 |
| SCADAScop | 1.8.1 |

## Highlights of this release

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

## Previous release

- **DNPScop Master** — a new **Inspect** tab in the Messages window sends one
  message at a time (link, class read, group read, time, assign class) with its
  parameters chosen inline; the scheduled list is now the **Communication Flow**
  tab. A link-layer **NACK / NOT_FUNCTIONING** is now bounded — reset and retried
  up to the configured count, each after the ACK timeout, then dropped — instead
  of retrying forever.
- **DNPScop Slave** — the fault-injection panel moved into its own
  **Communication Tests** window (right-click an RTU → *Communication Tests…*).
- **IEC104Scop / IEC101Scop Slave** — a **Communication Tests** tab injects
  application-, link- (and, for IEC 104, APCI-) layer faults; a negative
  confirmation now **rejects** the request rather than only flipping the P/N bit;
  **Send End of Initialization** added to the RTU menu.
- **ModbusScop Master / Slave** — resizable point / register-map columns with a
  right-click header menu (hide Name, show the address in the value cell, base-1
  register numbers).
- **SCADAScop** — open several **Communication Monitor** windows; closing an
  extra one removes it (the first stays and just hides).
- **All tools** — **Help → About** now links to this online manual, and the
  Toolbar and Status Bar show their names in the Ctrl+Tab window switcher.

The full change history for each release is on its GitHub release page.
