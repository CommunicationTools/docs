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
| DNPScop Master | 1.6 |
| DNPScop Slave | 2.5 |
| IEC104Scop Master / Slave | 1.6 / 1.6 |
| IEC101Scop Master / Slave | 1.6 / 1.7 |
| SCADAScop | 1.8 |

## Highlights of this release

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
