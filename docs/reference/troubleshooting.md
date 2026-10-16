# Troubleshooting

## The Master shows no data

- Is the channel **started** on both ends?
- Do the **addresses** match — master/outstation link address, common address,
  or unit id?
- For serial: same **baud / parity / stop bits**, and the right COM port?
- Open the **[Communication Monitor](../shared/monitor.md)**: are requests going
  out, and is anything coming back?

## Frames show but decode looks wrong

- Check the **framing** (Modbus RTU vs TCP/MBAP), the **address size** (IEC), and
  the link-address size (IEC 101).
- DNP3 rows reading *fragment cont. seg N* are the middle packets of a long
  message, not errors — see
  [Long DNP3 messages](../shared/monitor.md#long-dnp3-messages).
- A row tagged `[fault]` is a fault you injected on the Slave — clear it in
  **Communication Tests**.

## TLS does not connect

- Confirm the certificate/key and trust settings on both ends, and that the
  clocks are roughly right (certificate validity).
- The handshake result is logged to the status log and the monitor.

## A serial port disappeared / the TCP link dropped

- With **auto-reconnect** on, the tool reopens the link by itself with a back-off;
  watch the status log. Check the port still exists and nothing else holds it.

## A window is missing or off-screen

SCADAScop windows can live outside the main window (see
[Windows and monitors](../shared/windows.md)). **View → Bring all windows back**
moves every one of them back into the main window. A window whose monitor is no
longer connected comes back on its own at the next start. To keep every window
inside the main window, turn off **View → Windows outside the main window**.

## Capturing for a bug report

A short **[PCAP](../shared/pcap.md)** that reproduces the problem, plus the
workspace file, is the most useful thing to attach.
