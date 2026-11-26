# Release notes

Each tool is versioned independently and published on its GitHub release page
under [github.com/CommunicationTools](https://github.com/CommunicationTools). Use
**Help → Check for updates…** in a tool to see whether a newer release exists
(see [Updates](shared/updates.md)).

## Current versions

| Tool | Version |
|---|---|
| ModbusScop Master | 2.8 |
| ModbusScop Slave | 2.8 |
| DNPScop Master | 1.9 |
| DNPScop Slave | 2.9 |
| IEC104Scop Master / Slave | 1.9 / 1.8 |
| IEC101Scop Master / Slave | 1.9 / 1.10 |
| SCADAScop | 3.0 |
| scopctl | 0.4 |

Each tool is published in three builds — **Windows 10 / 11** (`win64`),
**Windows 7 SP1** (`win7`) and **Linux x86_64** (`linux-x86_64`); see
[Install](getting-started/install.md) and [Running on Linux](getting-started/linux.md).

!!! note "The standalone tools are frozen"
    New features are developed in **SCADAScop**, which contains every tool; the
    standalone tools remain available and receive fixes (DNPScop Slave 2.9 is
    such an upload: a feature of the shared DNP3 core).

## SCADAScop 3.0

- **SNMP Master and SNMP Slave** (SCADAScop only): SNMPv1, v2c and v3 — USM
  authentication MD5 / SHA-1 / SHA-224 / 256 / 384 / 512, privacy DES /
  AES-128 / 192 / 256 (with the "-C" key extension). **New Channel › SNMP**,
  role *Master* or *Slave*. See [SNMP](protocols/snmp.md).
- **Objects without a MIB.** A device's objects, tables, notifications and
  named types are kept complete in the workspace and edited by hand; **Import
  MIB…** copies a MIB's objects in, **Export MIB…** writes the device as an
  SMIv2 module, and the slave's **Import walk…** turns an `snmpwalk` capture or
  `.snmprec` file into a simulated device.
- **SNMP Slave**: simulated agents with editable and simulated values,
  communities, views and v3 users, traps and informs, Communication Tests
  (deliberate faults) and Lua RTU scripting with `notify()`.
- **SNMP Master**: several devices per channel, poll groups, SET, a Request
  window, an Events window for traps and informs, and **Device Discovery** —
  walk what a device has (Quick / Standard / Deep / Custom profiles) and import
  it into the device (**Discover (walk)…** in the device's Objects window and
  right-click menu); **Import walk…** reads a saved walk into a master device too.
  Values reach displays, the Watch window and charts.
- **Slave → master copy**: a copied slave device pasted under a master channel
  brings its definitions, the agent's address / version / credentials, and is
  polled at once. The standard SNMPv2-MIB scalars are named without a MIB file.
- **From MIB Library…** imports a loaded module's objects into a device;
  **Delete objects…** deletes ticked objects; Device Discovery ticks cascade
  through the tree, with Tick all / Untick all.
- **Row selection and bulk edits** in the SNMP Scalars tabs, as in the other
  protocols (Select all, Clear, Delete selected; a change on a selected row applies
  to the whole selection).
- **MIB Library**: right-click a module › Remove from library; a Files list with
  Remove / Restore; files loaded by hand are kept across restarts.
- **HCNUM-TC built in** (RFC 2856): MIBs using `CounterBasedGauge64` /
  `ZeroBasedCounter64` load without that file.
- **Tools › SNMP MIB Library**: your MIB folders, the modules found and an OID
  browser. No MIB files ship with the tool.

## SCADAScop 2.1 / DNPScop Slave 2.9

- **DNP3 slave — max points per object group.** Two RTU settings cap the
  points under one object header, for masters that limit what they accept per
  header. **Settings… › Application / Command** now has a **Message** section
  with *Max points per object group* — every response (class poll, group read,
  Class 0 scan) carries at most that many points of one group / variation
  under one header; a longer static run is split into ranges of that length,
  events into several headers in buffer order (`0` = unlimited, as before) —
  and a **Control** section with the previous settings. On the **Unsolicited
  and Event buffer** tab, under *Null unsolicited on start-up*, a checkbox and
  value give unsolicited messages their own limit; unchecked they follow the
  general one. The extra headers count against the Application Layer Max
  Message Size: what does not fit waits for the next poll. See
  [Points per object group](protocols/dnp3.md#points-per-object-group).
  Workspace keys `MaxPerObj`, `UnsMaxPerObjOn`, `UnsMaxPerObj`.
- **Soft Dark theme.** **View › Theme** has a fourth theme: neutral mid-dark
  greys instead of near-black, grey buttons that take the accent only when
  hovered or pressed, neutral docked tabs and a toned-down accent (the default
  Blue becomes steel blue). Its colours can be overridden in **Theme ›
  Settings…** like the other three. See [Themes](shared/themes.md).
- DNPScop Slave 2.9 is a fix-level upload of the frozen standalone tool with
  this feature; SCADAScop 2.1 keeps scopctl 0.4.

## Last standalone release

One more upload of the frozen standalone tools with what their shared sources
gained during the SCADAScop 2.0 round. Everything below is already in SCADAScop
2.0; the SCADAScop-only features (HMI displays, Watch window, windows on several
monitors, scopctl) are not in the standalone tools.

- **ModbusScop Slave 2.8, DNPScop Slave 2.8, IEC104Scop Slave 1.8, IEC101Scop
  Slave 1.10** — **RTU scripting**: right-click an RTU › **Script…**, tick the
  **Tag** column of the points the script may use, then accept, reject or delay
  commands and setpoints, react to value changes and timers, and simulate
  breakers, bays, tap changers and load profiles with the built-in library. See
  [RTU scripting](shared/scripting.md) and the [Script API](reference/script-api.md).
- **ModbusScop Master 2.8** — one window per RTU with a tab per poll map;
  **value lists** (enumerations and bit names) shown in the polled cells and
  offered when writing; write maps keep the value of their last successful
  write.
- **ModbusScop Slave 2.8** — value lists; **command maps** (FC05 / 06 / 15 / 16
  blocks); **On coil write**; Words / Bits view of the PLC memory; New Channel
  creates a first RTU.
- **DNPScop Master 1.9** — **Import to RTU** in the Live Data Map adds every
  received point the Points map does not have yet.
- **IEC104Scop / IEC101Scop Master 1.9** — Channel Settings are greyed while the
  channel runs; a control or setpoint row shows the value last executed by the
  RTU (positive ACTCON).
- **IEC104Scop Slave 1.8 / IEC101Scop Slave 1.10** — the **Communication Tests**
  window replaces the fault-injection panel of the Points window; Channel
  Settings are greyed while the channel runs.
- **All master tools** — a **Tag** column names points for SCADAScop's displays,
  scripts and Watch window. **All tools** — the About text no longer shows a
  garbled user-folder path.

## SCADAScop 2.0

New features are developed in SCADAScop only; this is the first release built on
that rule, and its two headline features earn it a major version.

- **RTU scripting** — every slave RTU (DNP3, IEC 104, IEC 101, Modbus) can run a
  Lua script: tick the **Tag** column of the points it may use, then decide
  commands and setpoints (accept, reject with the protocol's own refusal, or
  answer after a delay), react to value changes and timers, and simulate
  breakers, bays, tap changers and load profiles with the built-in library.
  Right-click an RTU › **Script…**. Scripts can also be started, stopped and
  reloaded from the web interface and `scopctl`. See
  [RTU scripting](shared/scripting.md) and the [Script API](reference/script-api.md).
- **HMI displays** — **Tools › Displays**: operator screens with values, LEDs,
  bars, gauges, breakers, power flows, images (PNG / SVG), command buttons and
  setpoint fields, bound to any master or slave point by tag. Commands follow
  select-before-operate. A display can belong to the whole workspace or to one
  RTU; export it once and import it for every RTU of the same type, and a reload
  of the file updates them all. See [HMI displays](shared/displays.md).
- **Windows and monitors** — drag the Communication Monitor, charts, displays
  or point maps out of the main window onto a second monitor and dock them
  together there; **View › Desktops** opens empty windows of their own (with a
  normal title bar: maximize, snap) to dock into. Dialogs open over the window
  you are working in, and text follows each monitor's scaling. See
  [Windows and monitors](shared/windows.md).
- **Watch window** — **Tools › Watch**: read-only lists of points of any
  channel to keep an eye on during a test — value, decoded quality, time of the
  last update, age and number of changes, for every master and slave point.
  Add points from a picker, **Add to watch** in the masters' maps and the
  slaves' points, or by drag and drop. See [Watch window](shared/watch.md).
- **New startup animations** — the splash screen now rotates through ten
  animated styles, six of them new substation scenes: a single-line diagram
  energising bay by bay, three-phase waveforms with a phasor star, an RTU point
  scan, a GOOSE mesh, a sequence-of-events log with time sync and a protocol
  frame decoder. **Help › About** previews them all.
- **scopctl 0.4** — the terminal client gains `script status | start | stop |
  reload`. See [scopctl](shared/scopctl.md).
- **Modbus Master** — one window per RTU with a tab per poll map. **Modbus
  Slave** — *On coil write* chooses where an accepted FC05 / FC15 write is
  stored; the PLC memory window has a Words / Bits view.
- Also in this release: Modbus value lists (enumerations and bit names), Modbus
  Slave command maps (FC05 / 06 / 15 / 16 blocks), DNP3 Master *Import to RTU*
  from the Live Data Map, and the IEC 104 / 101 Slave **Communication Tests**
  window.

## Final standalone round (SCADAScop 1.9)

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

## Earlier release

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
