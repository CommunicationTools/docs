# Status Messages reference

The **Status Messages** window (**View → Status Messages**) is the tool's event
log: one line per thing that happened on a channel — connections coming and
going, protocol procedures, commands and their results, anything the peer did
that the tool had to refuse or work around. It keeps the last 500 lines. In
SCADAScop there is one window for every embedded tool, with a **Source**
column naming the tool the line came from.

This page lists the messages **about the protocol**, per tool, with what each
one means and what to look at. Frame-by-frame detail (every request and reply,
fault tags, decode notes such as `RETRY 1/2 in 500 ms`) is in the
[Communication Monitor](../shared/monitor.md), not here.

**Reading the tables**

- `<channel>` and `<rtu>` stand for the names you gave the channel and the RTU
  (outstation / station / unit). `<peer>` is the other end's address
  (`10.0.0.5:20000`, `COM3`). `<n>`, `<ms>` and so on are numbers.
- Most lines are prefixed `<channel>: ` (channel-level) or `<channel>/<rtu>: `
  (RTU-level); the DNP3 and IEC slaves use `<channel> / <rtu> (link <n>) - `.
  The prefixes are left out of the tables.
- Housekeeping lines (workspace saved / loaded, points imported or exported,
  settings updated, scripts bound) are self-explanatory and are not listed.

---

## Common messages (every protocol)

These come from the shared transport layer, so they read the same whatever
protocol the channel speaks. A master prints them as `<channel>: …`; a slave
as `<channel> - …`.

### Connection life cycle

| Message | Meaning |
|---|---|
| `connecting to <peer>` / `opening <port>` | The channel was started; the TCP connect (or serial open) is under way. |
| `connected` | The transport is open. The master's start-up sequence follows (DNP3: restart / time / integrity; IEC 104: STARTDT; IEC 101: link reset). |
| `connection failed` | The connect attempt did not succeed (nothing listening, wrong host / port, firewall, serial port missing). Without auto-reconnect the channel stops here. |
| `connection lost` | A connection that was up has dropped: the peer closed it, the cable or the TCP keep-alive gave up, the USB serial adapter disappeared. |
| `stopped` | The channel was stopped from the tool. |
| `<peer>: reconnecting in <ms> ms (attempt <n>)` | **Auto-reconnect** is on and the link is down; the next attempt is scheduled with the back-off shown (doubling from the minimum up to the maximum you configured). |
| `<peer>: connection re-established` | The auto-reconnect succeeded after at least one earlier connection. Masters restart their start-up sequence; IEC 104 starts from a fresh APCI link (sequence numbers, windows). |
| `Started <host>:<port>` / `listening on <peer>` | A slave channel is listening. |
| `Start failed: <reason>` / `start failed - <reason>` | The slave could not listen. Serial reasons add a hint: `(the port is probably open in another program)` or `(no such port)`; TCP reasons are the OS text (`address already in use`, …). |
| `master connected <peer>` / `master disconnected <peer>` | On a slave: a master opened / closed a connection. The slaves accept several masters at once; one line per master. |

### Local interface (TCP / UDP)

Only when a **local interface** address is set on the channel.

| Message | Meaning |
|---|---|
| `Local interface "<addr>" is not an IP address - not connecting` | The local address field does not parse. Fix it or clear it. |
| `Local interface <addr> cannot be used (<os error>)` | The address exists but cannot be bound (not an address of this machine, or Linux needs `CAP_NET_RAW` to bind the device). |
| `Local interface <addr>: <host> has no address of the same IP version` | The local interface is IPv4 and the host resolved only to IPv6 (or the reverse). |

### RS-485 direction control (serial)

Printed once when a serial channel opens, after the RTS mode in the channel's
connection options.

| Message | Meaning |
|---|---|
| `<port>: RTS on` | RTS is held asserted (powered adapters, "RTS always on" mode). |
| `<port>: RTS could not be set on this port` | The driver refused the RTS call; the adapter may switch direction by itself, or it is not an RS-485 port. |
| `<port>: RS-485 direction switched by the driver` | The port's own RS-485 mode is used (Windows / Linux driver support). |
| `<port>: RS-485 direction switched by RTS around every frame` | Toggle mode: the tool raises RTS to transmit and drops it after the turnaround delay. |

### TLS

TLS lines appear in Status Messages and, as a byteless `TLS - …` /
`TLS WARNING - …` / `TLS ERROR - …` row, in the Communication Monitor.
See [TLS](../shared/tls.md).

| Message | Meaning |
|---|---|
| `TLS established with <peer>: <version>, <cipher>, peer CN=<name> (issuer <name>)` | The handshake succeeded; the session's version, cipher and the peer's certificate are shown. `no peer certificate` means the other side sent none (a client that is not required to). |
| `TLS handshake with <peer> failed: <reason>` | The handshake failed — usually a certificate the other side does not trust, a name that does not match the expected peer name, an expired certificate (check the clocks) or a TLS-version / cipher mismatch. The connection is closed; with auto-reconnect it will be retried. |
| `TLS configuration: <reason>` | The channel's own certificate / key / CA files could not be loaded (wrong path, wrong password, key does not match the certificate). The channel does not start. |
| `TLS requested but this build has no OpenSSL (built without TLS)` | This executable was built without TLS support. |
| `the server certificate is NOT verified (lab mode)` | Warning on a master: *verify peer* is off, so any certificate is accepted. Fine on a bench, not in production. |
| `client certificates are NOT verified (lab mode)` | The same warning on a slave. |
| `clients without a certificate are accepted` | Warning on a slave: peer verification is on but a client certificate is not required. |

---

## Modbus Master

Modbus master lines are prefixed `Channel <channel>: ` (channel events) or
`<window>: ` (a poll window).

### Channel

| Message | Meaning |
|---|---|
| `<proto> ready for <host>:<port>` / `connecting to <host>:<port> ...` | The channel started (UDP is connectionless, so it is *ready* at once; TCP connects first). |
| `connected on <port>` | The serial port opened. |
| `failed to open <port>` | The serial port could not be opened (in use, missing). |
| `connect failed (no response)` | The TCP connect did not complete within the connect timeout and auto-reconnect is off. |
| `connection lost - reconnecting` | The link dropped and auto-reconnect is on; poll windows pause until `connection re-established`. |
| `disconnected (peer closed)` | TCP: the slave closed the connection and auto-reconnect is off. |
| `serial port lost` | The serial port vanished (adapter unplugged) and auto-reconnect is off. |
| `connection lost - <n> pending request(s) failed` | The link dropped while requests were outstanding; each of them is reported to its poll window as a transport error. |

### Responses that do not fit (RTU and TCP)

The core matches every reply to the request it answers. A reply that does not
fit is logged here and does not update any poll window.

| Message | Meaning |
|---|---|
| `response from unit <n> with no request pending (late reply?) - ignored` | RTU framing: a reply arrived while nothing was waiting — typically the answer to a request that had already timed out. Raise the response timeout, or lower the poll rate. |
| `response for unknown transaction 0x<nnnn> (late or duplicate) - ignored` | TCP / MBAP: the transaction id matches no pending request (late reply, or a slave that echoes a stale id). |
| `response from unit <n>, expected <m> - not the awaited reply (late?), still waiting` | RTU: the unit id differs from the request's. The reply is dropped and the request keeps waiting for its own. On a shared bus this is another slave's answer. |
| `response from unit <n>, expected <m> (transaction id matched - accepted)` | TCP: same mismatch, but the transaction id proves which request it answers, so it is accepted — a quirk of the slave worth knowing about. |
| `<reason> - not the awaited reply (late?), still waiting` | RTU: the reply's PDU does not match the request (`function code does not match the request`, `byte count does not match the requested quantity`, `echoed address does not match the request`, `response of the wrong length`, …). Dropped; the request keeps waiting. |
| `invalid response: <reason>` | TCP: the transaction id matched but the PDU is malformed for that request (same reasons as above). The request fails with this text. |
| `<n> byte(s) dropped - not a Modbus RTU frame (resynchronised)` | RTU: bytes in front of a valid frame were skipped — line noise, or a frame with a bad CRC. Repeated often: wrong baud / parity, or two masters on the bus. |
| `<n> byte(s) dropped - incomplete frame or bad CRC - dropped when the line went silent` | RTU: a frame the inter-frame silence cut short, or whose CRC failed. |
| `malformed MBAP header (protocol id / length) - connection closed` | TCP: the stream no longer parses as Modbus/TCP (protocol id ≠ 0 or an impossible length); the only safe recovery is to close and reconnect. Usually a non-Modbus device on that port, or RTU framing where TCP framing was expected. |

### Poll windows and commands

| Message | Meaning |
|---|---|
| `<window>: polling disabled after error (<text>)` | The window's **Disable on error** option is on and a poll ended in an error. `<text>` is the result: `no response` (timeout), `Exception <code>: <name>` (the slave answered an exception — *Illegal Data Address*, *Illegal Function*, *Slave Device Busy*, …), `invalid response: …`, or `Command exception <code>: <name>` for a write sent from the window. Fix the cause and start the window again. |
| `<window>: command <FC> -> addr <n>` | A write (double-click a cell) was sent. The result shows in the window's response field: `Command OK`, `Command exception <code>: <name>`, or `no response`. |
| `display command <what> sent` / `display command <what> -> <result>` | A write sent from an [HMI display](../shared/displays.md) and its result. |

---

## Modbus Slave

Slave lines are prefixed with the channel name. `<peer>` is the master's
address.

| Message | Meaning |
|---|---|
| `Master connected: <peer> on <channel>` / `Master disconnected: <peer> on <channel>` | A master opened / closed a TCP connection. |
| `<peer>: no request for <n> s - connection closed (indication timeout)` | The channel's **indication timeout** elapsed with no request from this master, so the slave closed the connection — a master that connected and went quiet. Raise the timeout or set it to 0 to keep idle connections. |
| `<peer>: malformed MBAP header - connection closed` | TCP: the bytes from this master stopped parsing as Modbus/TCP; the connection was dropped. Usually RTU framing sent to a TCP port, or a non-Modbus client. |
| `<peer>: <n> byte(s) dropped - not a Modbus RTU frame (resynchronised)` | RTU: garbage in front of a frame was skipped (noise, wrong baud / parity, a frame with a bad CRC). |
| `<peer>: <n> byte(s) dropped - incomplete frame or bad CRC - dropped when the line went silent` | RTU: a frame cut short by the inter-frame silence, or with a bad CRC. |
| `<peer>: response of unit <n> (another slave on the bus) skipped` | RTU on a shared bus: the slave saw another slave's *response* and ignored it, as it should. Only informational; if it appears for a unit that is one of your own RTUs, two devices share that unit id. |
| `RTU '<name>': unit id <n> -> <m>` | An RTU's unit id was changed. |

Requests the slave refused are visible in the Communication Monitor as
`Exception <code>` rows (and `no such unit (silent)` for a unit that does not
exist on the channel); they are not repeated here, since every request would
otherwise flood the log. Writes refused by an RTU script show up as the
exception the script chose.

---

## DNP3 Master

Channel lines are `<channel>: …`; outstation lines are `<channel>/<rtu>: …`.

### Responses, time-outs and retries

| Message | Meaning |
|---|---|
| `TIMEOUT - no response within <ms> ms` | The outstation did not answer a request inside the **response timeout**. The RTU goes offline in the tree until it answers again. With application-layer retries configured, the retries follow (they are shown in the monitor as `RETRY n/m in <ms> ms`); each retry resends the same request with the same sequence number, so an outstation that did answer repeats its response from history instead of executing the request twice. Check the link addresses, the timeout against the outstation's response time, and whether the outstation is up. |
| `outstation time <date time> (master <date time>)` | The answer to a **Read Time** (g50v1): the outstation's clock next to the master's. A large difference calls for a time sync. |

### Internal Indications (automatic reactions)

The master watches the IIN bits of every response and reacts once per response
that shows the bit (each reaction can be turned off in the RTU settings):

| Message | Meaning |
|---|---|
| `IIN1.7 restart - clearing it` | The outstation reports it has restarted; the master writes g80 to clear the bit. Expect to see the integrity poll re-run and, if the outstation lost its clock, `IIN1.4 needs time` next. |
| `IIN1.4 needs time - syncing` | The outstation asks for a time synchronisation; the master sends one (plain, or with delay measurement, after the RTU setting). |
| `IIN2.3 event buffer overflow - integrity poll` | The outstation's event buffer overflowed, so events were lost; the master runs an integrity poll to refresh its image. Recurring: the master polls too slowly for the event rate, or the outstation's buffer is too small. |

Class-event bits (IIN1.1 / 1.2 / 1.3) trigger a class poll silently.

### Time synchronisation

| Message | Meaning |
|---|---|
| `time sync - round trip <ms> ms, turnaround <ms> ms, writing now + <ms> ms` | Delay-measurement sync: DELAY_MEASURE was answered, the one-way delay was computed from the round trip and the outstation's turnaround, and the time is written with that offset. |
| `DELAY_MEASURE answered without a delay object - writing the time as is` | The outstation answered DELAY_MEASURE but sent no g52 object (not supported at its level); the time is written without compensation. |
| `DELAY_MEASURE unanswered - time sync dropped` | No answer to the delay measurement; the sync is abandoned this time. |
| `RECORD_CURRENT_TIME unanswered - LAN time write dropped` / `WRITE g50v3 unanswered` | LAN-style sync (g50v3): one of its two steps got no response. |

### Controls

| Message | Meaning |
|---|---|
| `command sent to BO <n>` / `command sent to AO <n>` | A Direct Operate (or a one-shot Operate) was sent. |
| `SELECT sent to BO <n> - OPERATE follows on its response` | Select-before-operate: the SELECT went out; the OPERATE is held until the outstation answers. |
| `SELECT unanswered - OPERATE dropped` | The SELECT got no response within the timeout, so the queued OPERATE was discarded — nothing was operated. |
| `command to Binary Output[<n>] -> <status>` / `command to Analog Output[<n>] -> <status>` | The outstation's answer to a control, with the DNP3 control status it returned: **Success**; **Timeout** (the outstation says the SELECT expired before the OPERATE); **No Select** (OPERATE without a matching SELECT); **Format Error**; **Not Supported / no such point**; **Already Active**; **Hardware Error**; **Local** (the outstation is in local mode); **Too Many Objects**; **Not Authorized** (refused by the outstation's permissions). |
| `display command SELECT to BO <n>` / `display command OPERATE sent` | The same, for a command issued from an [HMI display](../shared/displays.md). |

### Inspect

`sent <message>` (`sent RESET_LINK_STATES`, `sent REQUEST_LINK_STATUS`, `sent
Class 1 read`, …) confirms a one-shot message from the **Inspect** tab went
out; the answer is in the monitor.

---

## DNP3 Slave

Outstation lines read `<channel> / <rtu> (link <n>) - …`. Several masters can
talk to one outstation: each master address gets a **master slot** (1, 2, …)
with its own event buffer, sequence tracking and unsolicited state, which is
why most lines name the slot.

### Confirmations and request handling

| Message | Meaning |
|---|---|
| `master slot <n>: request FC <f> deferred until the CONFIRM<…>` | The outstation has a response with CON set outstanding (an unsolicited response, or a multi-fragment response) and is waiting for the master's application CONFIRM. A new request from that master (`FC <f>`; FC 1 is READ) arrived meanwhile. A READ cannot be answered until the pending events are confirmed or time out, so it is held and answered afterwards; `(replaces an earlier one)` means another request was already waiting and was dropped for this one. Harmless when the CONFIRM follows at once (a poll and an unsolicited crossing on the wire). If it is always followed by `no CONFIRM`, the master is not confirming unsolicited responses. |
| `master slot <n>: no CONFIRM - <e> event(s) back in the buffer<…>` | The confirm timeout expired on a solicited response that carried events; the events are returned to the buffer and will be sent again. `(pending fragments dropped)`: the rest of a multi-fragment response was discarded too. The master is not sending application CONFIRMs, or sends them after the outstation's timeout. |
| `master slot <n>: repeated request FC <f> answered from history (not executed again)` | The master sent the same request (same sequence number) again — its application-layer retry. The outstation replied with the stored response instead of executing it twice, as the standard requires. A retry means the master missed the first response: look at the master's timeout. |

### Unsolicited responses

| Message | Meaning |
|---|---|
| `master slot <n>: ENABLE_UNSOLICITED - classes now <list>` / `DISABLE_UNSOLICITED - classes now <list>` | The master enabled or disabled unsolicited reporting; the classes still enabled for that master are listed (`none` when all are off). |
| `master slot <n>: null unsolicited not confirmed - retry <k>` | The start-up null unsolicited response (the one that announces the outstation is up) was not confirmed; it is being resent. |
| `master slot <n>: unsolicited (<e> event(s)) not confirmed - retry <k>` | An unsolicited response carrying events was not confirmed; resent with the same events. |
| `master slot <n>: null unsolicited given up (no CONFIRM); will try again` | All retries of the null unsolicited failed; the outstation backs off and tries again later. The master is not confirming — or there is no master at that address. |
| `master slot <n>: unsolicited given up - <e> event(s) back in the buffer` | All retries of an event unsolicited failed; the events go back to the buffer for the next unsolicited or the next class poll. |

### Functions executed for the master

| Message | Meaning |
|---|---|
| `master slot <n>: time written (g50v1) <date time> UTC - NeedTime cleared` / `(g50v3 LAN)` | The master synchronised the outstation clock (plain write, or the LAN procedure RECORD_CURRENT_TIME + g50v3). IIN1.4 is cleared. |
| `master slot <n>: COLD_RESTART - IIN1.7 asserted to every master (nothing restarts in the tool)` / `WARM_RESTART - …` | A restart request was accepted: the tool only raises the restart bit for every master (so each of them runs its restart handling); nothing actually restarts. |
| `master slot <n>: ASSIGN_CLASS - every g<g> point -> class <c>` / `ASSIGN_CLASS - g<g> <a>..<b> -> class <c>` | The master reassigned the event class of a group (or a range of it). |
| `master slot <n>: FREEZE_AT_TIME - counters freeze once at <time>` / `…, then every <ms> ms` | A scheduled counter freeze was accepted; `Freeze-at-time: <n> counter(s) frozen (repeating)` follows when it fires. |
| `<rtu>: function 0x<ff> not supported` | A request with a function the outstation does not implement was answered with IIN2.0 (*function not supported*). |
| `<rtu>: COLD_RESTART not supported (IIN2.0 set)` / `WARM_RESTART not supported (IIN2.0 set)` | Cold / warm restart support is turned off in the outstation's settings; the request was answered with IIN2.0. |
| `<rtu>: ENABLE_UNSOLICITED not supported at Level <l> (IIN2.0 set)` / `DISABLE_UNSOLICITED …` | The outstation emulates a subset level without unsolicited support (Level 1); answered with IIN2.0. |

### Controls received

| Message | Meaning |
|---|---|
| `Control: BO <n> SELECT refused (<reason>)` / `Control: AO <n> SELECT refused (<reason>)` | A SELECT was refused. `<reason>` is the point's permission (`selection denied`, `control not allowed`) or `refused by script, status <s>` when the RTU script rejected it. |
| `Control: BO <n> refused (<reason>) - NOT_AUTHORIZED` / `Control: AO <n> refused …` | An OPERATE / DIRECT OPERATE was refused with status NOT_AUTHORIZED, for the same reasons (`execution denied`, `control not allowed`). |
| `Control: BO <n> refused (refused by script) - status <s>` | The RTU script rejected the operate with the status it chose. |
| `Glue: BO <n> operate -> <k> action(s) run` / `Glue: AO <n> setpoint <v> -> <k> action(s) run` | The control was executed and its glue-logic rule ran `<k>` actions. |
| `Glue: BO <n> operate -> nothing ran (<why>)` | Executed, but no glue action ran: `no glue rule configured`, `rule disabled`, `rule has no trigger groups`, `no trigger group matches this operate`, `rule has no actions`, `action target point does not exist`. |
| `Glue: BO <n> operate rejected (status <s>) - glue not run` | The control itself was rejected (status ≠ 0 from the Communication Tests settings), so no glue ran. |
| `Script: BO <n> operate accepted` / `Script: AO <n> setpoint <v> accepted` | The RTU script accepted the control (and handles it itself). |

---

## IEC 60870-5-104 Master and IEC 60870-5-101 Master

Both masters share one tool, so most lines are common; the link-layer lines
belong to -101 and the STARTDT lines to -104.

### Connection and start-up

| Message | Meaning |
|---|---|
| `connecting to TCP <host>:<port>` / `connecting to TLS <host>:<port>` | -104 channel started. |
| `opening <port>` | -101 serial channel started. |
| `connected` | The transport is open. |
| `STARTDT confirmed - data transfer started` | -104: the outstation confirmed STARTDT; I-frames may now flow. |
| `STARTDT confirmed - interrogating` | -104, with **GI on start**: a general interrogation was sent to every enabled station. |
| `STOPDT confirmed - data transfer stopped` | -104: the tool stopped data transfer. |
| `closed - <reason>` | -104: the APCI link closed the connection for a protocol reason: `t1 timeout - no acknowledge` (the outstation did not acknowledge within t1, or did not answer TESTFR), `N(S) does not match V(R)`, `N(R) outside the sent window` (sequence errors — only fatal when *strict sequence* is on), `S-frame in the stopped state`, `I-frame before STARTDT` (only with *strict start*). The connection is reopened by auto-reconnect when enabled. |
| `reconnecting in <ms> ms` | Auto-reconnect is scheduled. |
| `link established - interrogating` | -101, with **GI on start**: a station's link came up (REQUEST STATUS + RESET LINK answered), and a general interrogation was sent to it. |
| `end of initialisation` / `end of initialisation - interrogating` | The station sent M_EI_NA_1: it (re)started. With GI on start the image is refreshed with a new interrogation. |

### -101 link layer

Prefixed `link <addr>: ` (the station's link address).

| Message | Meaning |
|---|---|
| `link up` | REQUEST STATUS and RESET LINK were answered; user data can flow. |
| `link down - <why>` | The link was lost: `no reply` (the retries of one request ran out), or the station answered the start-up with a negative link function (`LINK NOT FUNCTIONING`, `SERVICE NOT IMPLEMENTED`). The link restarts from REQUEST STATUS after a back-off. Check the link address and its size, baud / parity, and whether the station is there. |
| `no answer - repeating the request (attempt <n>)` | A request got no acknowledge within the timeout and is being repeated (the FCB is kept, so the station can tell it is a repeat). |
| `repeated frame (same FCB) - the last reply sent again` | Balanced mode: the station repeated a frame to us (it missed our acknowledge); we resent our stored reply. |
| `RESET OF REMOTE LINK - frame count bit restarted` | Balanced mode: the station reset our secondary direction. |
| `RESET OF USER PROCESS - <n> queued ASDU(s) discarded` | Balanced mode: the station reset the user process; our queued user data was dropped. |
| `buffers full (DFC = 1) - user data refused / held` | The station signals data-flow control; queued commands are held until `buffers free again (DFC = 0)`. |
| `incomplete frame dropped after the line went idle (<n> byte(s))` | A frame was cut short (noise, wrong baud / parity, link-address size mismatch). |

### Commands

| Message | Meaning |
|---|---|
| `SELECT <kind> <ioa> - awaiting confirmation` | A select was sent (SBO); the execute is held. |
| `EXECUTE <kind> <ioa>` | An execute (or a direct command) was sent. |
| `<type> -> activation con` | The station's confirmation of a command or a system function (interrogation, clock sync, counter interrogation, …). |
| `<type> -> activation con (NEGATIVE)` | The station refused it: unknown IOA, wrong type for the point, local mode, interlock, time tag outside the window… Look at the station's own log for the reason. |
| `<type> -> activation term` | The station reports the command completed. |
| `<type> -> deactivation con` | Confirmation of a deactivation (cancelled command / stopped cyclic function). |
| `<type> -> unknown type` / `unknown cause` / `unknown common address` / `unknown IOA` | The station could not interpret the ASDU (COT 44–47): the type is not supported, the common address or the IOA does not exist on that station. Check the RTU's common address and the point list. |
| `selection confirmed -> EXECUTE <label>` | SBO: the positive confirmation of the select released the execute. |
| `selection denied - execute cancelled (<label>)` | SBO: the select was answered negatively; the execute was not sent. |
| `selection timed out - execute cancelled (<label>)` | SBO: no confirmation for the select within the window; the execute was not sent. |
| `display command SELECT <kind> <ioa>` / `display command EXECUTE sent (IOA <n>)` | Commands issued from an [HMI display](../shared/displays.md). |

`<type>` is the ASDU type name, e.g. `C_SC_NA_1 (single command)`,
`C_IC_NA_1 (interrogation command)`; `<kind>` is the point kind (`SP`, `DP`,
`SC`, `SE`, …).

---

## IEC 60870-5-104 Slave and IEC 60870-5-101 Slave

Station lines read `<channel> / <rtu> - …` (`<rtu>` is the station name, or
`CA <n>`).

### Connections and link

| Message | Meaning |
|---|---|
| `listening on <peer>` (-104) / `opened <port>` (-101) | The channel started. |
| `start failed - <reason>` | Could not listen / open the port. |
| `master connected <peer>` / `master disconnected <peer>` | -104: a master opened / closed a connection. |
| `link <addr>: …` | -101 link-layer events — the same set as on the master side (above): `link up` / `link down - no acknowledge` / `link down - no answer to the link start-up` (balanced mode), repeats, `RESET OF REMOTE LINK`, `RESET OF USER PROCESS`, DFC, resync. |

A -104 connection that the station closes for a protocol reason (sequence
error, t1 time-out, a second master superseding the active one in a redundancy
group, a master not in the allowed list) is noted in the Communication
Monitor as `<reason> - closing connection`.

### Requests handled

| Message | Meaning |
|---|---|
| `Interrogation answered (<n> points, <m> point db)` / `Group <g> interrogation answered (<n> points)` | A general (or group) interrogation was answered with `<n>` points. |
| `Interrogation QOI <q> not supported` | The qualifier is not 20 (station) or a group 21–36. Answered negatively. |
| `Counter interrogation (<what>): <n> counter(s)` / `Counter interrogation RQT <r> not supported` | Counter interrogation answered / refused. |
| `Clock synchronized to <time>` | C_CS_NA_1 accepted; the station clock now follows the master. |
| `Clock synchronization with an invalid time tag - clock left as is` | The clock sync carried an invalid (IV) time; ignored. |
| `End of initialisation queued (M_EI_NA_1, COI <c> …)` | The station is sending its end-of-initialisation (after start or a reset process). |
| `Reset process (QRP <q>): <n> pending event(s) dropped<…>` / `Reset process QRP <q> not supported` | C_RP_NA_1 handled / refused. |
| `Parameter <type> for IOA <n> = <v> (QPM 0x<..>)` / `Parameter activation for IOA <n> (QPA <q>) activated` / `Parameter for IOA <n> refused (unknown monitor point)` | Parameter of measured value / parameter activation handled or refused. |
| `Type <t> with <n> objects refused (one allowed)` | A command ASDU carried more than one object; refused. |

### Commands received

| Message | Meaning |
|---|---|
| `Control: <kind> <ioa> selected` | A SELECT was accepted (SBO). |
| `Control: <kind> <ioa> executed, value <v>` | A command was executed. `(accepted by script)` when the RTU script took it. |
| `Control: <kind> <ioa> SELECT refused (<reason>)` / `EXECUTE refused (<reason>)` | Refused by the point's permission: `selection denied`, `execution denied`, `control not allowed`. Answered with a negative confirmation. |
| `Control: <kind> <ioa> SELECT refused by script (<code>)` / `EXECUTE refused by script (<code>)` | The RTU script rejected the command. |
| `Control: command <ioa> deactivated` | A deactivation (COT 8) was received for the command. |
| `Command to IOA <n> refused (unknown object)` | No control point with that IOA on this station — answered with COT 47 (unknown IOA). |
| `Command to IOA <n> refused: time tag <ms> ms from the station clock (window <ms> ms)` | A time-tagged command whose tag is outside the acceptance window (stale or future command). Check the master's clock sync. |
| `Glue: command <ioa> -> <k> action(s) run` | The glue-logic rule bound to the command ran. |
| `Link: <kind> <ioa> <- <v> (from command <ioa>)` / `Link: target IOA <n> missing or not a monitor point` | A command linked to a monitor point updated it / could not. |
| `[fault] …` | A fault you injected in **Communication Tests** was applied to this reply (`rejected with COT 44`, `negative confirmation`, `dropped the activation confirmation`, `dropped the activation termination`, `wrong common address`). |

---

## P2P monitor

The P2P channel sits between a master and an RTU; its lines are prefixed with
the channel name and, for per-master lines, the master's address.

| Message | Meaning |
|---|---|
| `listening on port <n> (<protocol>[, TLS towards the master][, TLS towards the RTU])` / `serial line <port> open (<protocol>…)` | The master side is open. |
| `cannot listen on port <n> - <reason>` / `cannot open serial port <port> - <reason>` | The master side could not open. |
| `<master>: master connected` / `master disconnected` / `serial line open` / `serial line closed` | A master attached to / left the master side. |
| `<peer>: refused (master limit)` | More masters than the channel allows; this one was turned away. |
| `<master>: connecting to RTU <peer>` | The RTU leg is being opened for this master. |
| `<master>: waiting for the first bytes before connecting to RTU <peer>` | Serial master leg in *connect on first data* mode: the RTU connection is made when the master first transmits. |
| `<master>: RTU <peer> connected` | The RTU leg is up; frames are relayed. |
| `<master>: RTU connection failed` / `RTU connection lost` | The RTU leg could not be opened / dropped. The master is left connected; its requests get no answer until the RTU leg is back. |
| `<master>: cannot open the RTU serial port <port>` / `cannot open the RTU connection` | The RTU leg could not even start. |
| `<master>: device <peer> connected (gateway)` / `device connection lost (gateway)` / `device connection failed (gateway)` | The same, for the Modbus gateway mode (TCP master ↔ RTU device). |
| `<master>: gateway: unit <u> FC <f>: <why> - Exception 0x0A (gateway path unavailable)` | Modbus gateway: the request could not reach the device (`device side down`, `device side not connected`, `gateway queue full (<n>)`); the master got exception 0x0A. |
| `<master>: gateway: unit <u> FC <f>: no answer within <ms> ms - Exception 0x0B (gateway target failed to respond)` | Modbus gateway: the device did not answer; the master got exception 0x0B. |
| `<master>: gateway: malformed MBAP header from the master - <n> byte(s) dropped` / `malformed MBAP header from the device - dropped` | Modbus gateway: unparsable Modbus/TCP on one side. |
| `<master>: gateway: device answer for transaction <n> matches no pending request - dropped` / `answer from unit <u> matches no pending request - dropped` | Modbus gateway: a late or unexpected reply from the device. |
| `master side: <note>` / `<master>: RTU side: <note>` | A transport note (local interface, RS-485 RTS mode, TLS event) on either leg — the texts are in [Common messages](#common-messages-every-protocol). |
| `stopped` | The channel was stopped. |

---

## Where to look next

- A message you do not expect to see repeatedly (`TIMEOUT`, `no CONFIRM`,
  `link down`, `closed - t1 timeout`) is best read next to the frames around it
  in the [Communication Monitor](../shared/monitor.md); a short
  [PCAP](../shared/pcap.md) of it is the thing to attach to a bug report.
- [Troubleshooting](troubleshooting.md) walks through the usual causes —
  addresses, framing, serial settings, TLS trust.
