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
- A row tagged `[fault]` is a fault you injected on the Slave — clear it in
  **Communication Tests**.

## TLS does not connect

- Confirm the certificate/key and trust settings on both ends, and that the
  clocks are roughly right (certificate validity).
- The handshake result is logged to the status log and the monitor.

## A serial port disappeared / the TCP link dropped

- With **auto-reconnect** on, the tool reopens the link by itself with a back-off;
  watch the status log. Check the port still exists and nothing else holds it.

## Capturing for a bug report

A short **[PCAP](../shared/pcap.md)** that reproduces the problem, plus the
workspace file, is the most useful thing to attach.
