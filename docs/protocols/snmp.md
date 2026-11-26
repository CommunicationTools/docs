# SNMP

SCADAScop 3.0 adds the **SNMP Master** (a manager: polls agents, writes with
SET, receives traps and informs, discovers what a device has) and the **SNMP
Slave** (simulated agents: answer managers from an object database you build
by hand, from a MIB or from a recording of a real device). Both speak
**SNMPv1, SNMPv2c and SNMPv3** (USM: MD5, SHA-1 / 224 / 256 / 384 / 512
authentication; DES, AES-128 / 192 / 256 privacy, including the "-C" key
extension used by Cisco devices). They exist only inside SCADAScop.

New Channel › Protocol **SNMP (v1 / v2c / v3)**, then Role *Master* or *Slave*.

## Objects — the device's database

SNMP gives a manager nothing until it asks, and asks by **OID**. What a device
"has" is therefore a list of object definitions, and both tools keep it in the
workspace, complete: name, OID, syntax (base type or a named type, with
enumerations, ranges, sizes), access, units, description, default value, and for
tables the entry with its INDEX. A MIB file is a source to import from and a
target to export to — never needed at run time: a workspace opens on a PC that
has none of the MIBs.

The **Objects** window (right-click the device › Objects…) has tabs:

| Tab | Content |
|---|---|
| **Scalars** | one row per instance (`<object>.0`): name, syntax, access, value, simulation, units, tag |
| **Tables** | the tables on the left; the selected one as a grid, one row per index, one column per column object |
| **Notifications** | the device's notifications; on the slave also the notification targets, *Send now* and the log of what was sent |
| **Types** | named types (textual conventions): an enumeration or a refined type several objects share |

Ways to fill it:

- **Add object… / Add table… / Add type…** — the manual editors. Names follow the
  SMI rules (start lower-case; letters, digits, hyphens). *next free* proposes the
  next OID under the same parent. A table needs at least one index column; tick
  *Managers can create / delete rows* to add a `RowStatus` column.
- **Import MIB…** — pick a MIB file; the modules it imports are looked up in its
  folder (and the folders you add). The dialog lists the diagnostics and the
  module's scalars, tables and notifications with check boxes; the selection is
  copied into the device with the types it uses.
- **From MIB Library…** — the same dialog on a module already loaded in
  **Tools › SNMP MIB Library** (pick it in the Module list); no file to open.
- **Selecting rows** (Scalars tab, slave and master) — click, Ctrl+click,
  Shift+click, drag a box, Ctrl+A, Escape; Ctrl+C copies name / OID / value.
  **Select all**, **Clear** and **Delete selected** are above the table (the master
  also has **Read selected now**). With several rows selected, a change on one of
  them applies to all: on the slave the value (instances of the same type and
  syntax), Sim, tag on and Notify on change; on the master the poll group, tag on
  and the row menu's Poll group / Read now.
- **Delete objects…** — every scalar, table and notification of the device with a
  check box (filter, All / None); the ticked ones are deleted after a confirmation,
  with their values (and, on the master, their polling). One object at a time:
  right-click its row › Delete.
- **Discover (walk)…** (master) — Device Discovery: the master walks the agent and
  shows what it has as a live tree and tables with values; tick objects or tables
  and **Import to device** adds their definitions and polls them. A tick on a branch
  covers everything under it (the rows below show ticked; a branch with only part
  ticked shows a mixed mark); unticking one row inside a ticked branch keeps the
  rest ticked. **Tick all** / **Untick all** are above the tree. Also in the
  device's right-click menu. A device with no objects says so at the top of its
  Objects window.
- **Import walk…** (master and slave) — an `snmpwalk` capture (`-On`, `-One` or the default
  named form) or an `.snmprec` file: every value becomes an instance, with
  definitions from the MIB loaded in Import MIB when it knows the object, inferred
  from the values otherwise (the slave uses the MIB Library when no MIB file was
  opened in Import MIB). On the slave a recording of a real device becomes a
  simulated one; on the master the values tell it what to poll.
- The standard scalars every agent has — the SNMPv2-MIB `system` group
  (`sysDescr`, `sysUpTime`, `sysName` …, 1.3.6.1.2.1.1) and the `snmp` counters
  (1.3.6.1.2.1.11) — are built into the slave and named without a MIB file.
- **Export MIB…** — writes the device's objects under a root OID as an SMIv2 module
  (IMPORTS, MODULE-IDENTITY, textual conventions, tables, notifications and
  conformance groups); the file is read back by the tool's own MIB loader before it
  is saved. It exports *your* objects: put them under your own subtree
  (1.3.6.1.4.1.&lt;enterprise number&gt;…) and use that as the root. The standard
  points (system / snmp groups) belong to SNMPv2-MIB, which every manager already
  has, so they are not exported.

Copy / Export device and Paste work between the two tools: the definitions are the
same rows on both sides, so a device polled by the master can be pasted under a
slave channel to simulate it, and the reverse. A slave device pasted into a master
channel brings its definitions (the standard scalars included), the agent's
address, port, version, communities and first v3 user, and every readable object
is polled.

## SNMP MIB Library

**Tools › SNMP MIB Library** holds your MIB folders and shows the modules found
(ok / partial / errors, with their messages) and a browser of the OID tree with
search by name or OID. It does not change a device by itself; it is used by:

- the SNMP Master's names for OIDs the device has no definition for (Request
  window, Events, Device Discovery);
- Device Discovery and **Import walk…**, which take names, types and tables from it;
- **From MIB Library…** in a device's Objects window, which copies a loaded module's
  objects into the device.

The window's *How the library is used* header says the same.

To take a MIB out of the library, right-click its module in the module list ›
**Remove from library**. A file loaded with **Load file…** is dropped; a file found
in one of your folders is left out of the library but stays on disk. The **Files**
header lists both, with **Remove** and **Restore**. Removing a folder (its **x**)
drops all of its modules. These settings are kept with the folders. No MIB file ships with SCADAScop; the core SMI modules every MIB
imports from (SNMPv2-SMI, SNMPv2-TC, SNMPv2-CONF, RFC1155-SMI, RFC-1212, RFC-1215)
are built in, and so are fallbacks for RFC1213-MIB and HCNUM-TC (RFC 2856:
`CounterBasedGauge64`, `ZeroBasedCounter64`), used when no file of that name is in
your folders. A `CounterBasedGauge64` object is a Counter64 on the wire with gauge
meaning: the slave lets it simulate as Sine or Ramp, not only as a counter.

## SNMP Slave

One channel is one UDP endpoint serving **one device**. The default port in the
New Channel dialog is 1161: ports below 1024 need administrator rights, and
Windows may already run its own SNMP service on 161.

- **Values**: double-click a value to edit it (octet strings as text, or `0x` +
  hex; enumerations from a list). Values written by a manager flash.
- **Simulation**: per instance — Sine, Ramp, Random, Counter (counts up at a rate
  and wraps like Counter32 / Counter64), Toggle, Uptime — driven by the shell's
  Simulation button and speed. `sysUpTime` always runs.
- **Access** window: the SNMP versions answered, the **communities** (read-only or
  read-write, each with an optional view), the **views** (subtrees included /
  excluded, with an optional mask; the most specific one decides) and the
  **SNMPv3 users** (authentication / privacy protocols and passphrases, the lowest
  security level accepted, write access, view). *cmd* copies the matching net-snmp
  command line. The engine ID is generated at the first start and kept in the
  workspace; boots count up on every start.
- **Notifications**: targets (host, port, version, community or v3 user, trap or
  inform with timeout / retries), `coldStart` when the channel starts,
  `authenticationFailure` while `snmpEnableAuthenTraps.0` is enabled (managers can
  set it), *Send now* per notification, and *Notify on change* on a scalar's
  right-click menu. Informs are retried until acknowledged; v3 informs discover
  the receiver's engine.
- **Communication Tests** (right-click the device): no reply, forced error-status,
  wrong request-id, duplicated, truncated or corrupted responses, oversize
  responses, GetNext that repeats or goes backwards, and for v3 a wrong digest,
  engine time skew, boots reset, wrong engine ID or a downgraded security level —
  each Off / Always / Once / Every Nth, for the request types ticked.
- **Script…**: an RTU script like the other slaves ([RTU scripting](../shared/scripting.md)).
  Tick *Tag* on the objects it uses. A manager's SET of a tagged writable object
  asks `on_setpoint` (numbers) or `on_command` (enumerations, `cmd.state` is the
  label) first; `REJECT("WRONG_VALUE")` answers with that error-status (unknown
  codes: `inconsistentValue`), `return ACCEPT, 1500` holds the answer.
  `notify("name", { tag = value })` sends one of the device's notifications.
  Events: `master_online` / `master_offline` (a manager counts as online while
  its requests arrive within 60 s), `sim_on` / `sim_off`.

## SNMP Master

One channel is one local UDP socket polling **any number of devices** (each a
host, port, version and credentials). The Device Settings dialog switches its
security section with the version; *Test* reads `sysDescr.0` and `sysUpTime.0`
with the settings entered and shows the result or the exact failure (timeout,
the Report name, the error-status).

- **Objects**: the same tabs as the slave, with live values, quality and age;
  poll groups set how often each object is read (scalars by GET, tables by
  walking their columns). Right-click: Write value (SET), Read now, Add to chart,
  Add to watch.
- **Request** window: Get / GetNext / GetBulk / Set / Walk of typed OIDs or names
  — the tool's `snmpget`.
- Polled values are on the shell's tag bus (`snmp:<channel>/<device>/o/<OID>`):
  displays, the Watch window and charts use them; a display's command on a
  writable object is a SET.
- A device that does not answer is shown **offline** and probed with
  `sysUpTime.0`; a v3 credential problem shows **auth error** with the Report name.

## Communication Monitor

Every datagram is a row (`v2c GetResponse id=42 3 vb`, `v3 Report
usmStatsNotInTimeWindows`, …). The decoded tree shows the envelope, the community
as it is on the wire, the v3 header and USM parameters, and — for encrypted v3
messages of users the tool knows — the **decrypted** ScopedPDU. Passphrases and
keys never appear anywhere.

## Testing against net-snmp

The tools were tested against the net-snmp tools (`snmpget`, `snmpwalk`,
`snmpbulkwalk`, `snmpset`, `snmptrap`, `snmpinform`, `snmpd`, `snmptrapd`):

```
snmpwalk -v2c -c public 127.0.0.1:1161 .1
snmpget -v3 -l authPriv -u scopuser -a SHA-256 -A <auth> -x AES -X <priv> 127.0.0.1:1161 1.3.6.1.2.1.1.1.0
snmptrapd -f -Lo -c trapd.conf udp:1162
```
