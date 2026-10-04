# P2P Monitoring (man-in-the-middle)

A **P2P Monitoring** channel sits *between* a master (front-end) and its RTU and
forwards every byte untouched in both directions, while showing — and optionally
recording — the traffic that passes through. It turns the tool into a passive
tap on a live link, with no change to either end.

It is created in **SCADAScop**: **Channels → + P2P channel**.

## What it is for

- **Watch a live link without Wireshark.** Insert the proxy between a front-end
  and a device and read the traffic, decoded, in a
  [Communication Monitor](monitor.md) — handy on locked-down machines where you
  cannot install a packet sniffer.
- **Terminal server / media converter.** Each leg is independently **TCP** or a
  **serial COM port**, so the proxy can receive an IP connection and forward it
  to a serial device, or the reverse, as well as TCP↔TCP or serial↔serial:
    - TCP front-end ↔ TCP device
    - TCP front-end ↔ serial device  (receive on IP, forward to the line)
    - serial front-end ↔ TCP device  (receive on the line, forward to IP)
    - serial front-end ↔ serial device
- **Capture to PCAP.** Everything that flows through can be written to a
  Wireshark-ready [`.pcap`](pcap.md) for later analysis.

## Decoding

The monitor decodes the traffic with the family decoders — **DNP3**,
**IEC 60870-5-101**, **IEC 60870-5-104**, **Modbus/TCP** and **Modbus RTU** — or
shows it as **raw bytes** when you only need to see the wire. Pick the protocol
when you create the channel.

## How it connects

A P2P channel is defined by the side it **listens** on and the side it **connects**
to (`listen → device ip:port`, or a COM port on either leg). It forwards as soon
as both ends are up; for a serial front-end leg the device connection is opened
when the port opens or on the first bytes (*Connect on first data*). One master
at a time per channel — a serial line has a single peer.

## Sitting inside a TLS link (SSL strip)

Each leg can run **TLS independently**: the proxy is a **TLS server toward the
master** and a **TLS client toward the RTU**. With the right certificates it can
therefore sit *inside* an encrypted link and present the traffic in clear text in
the monitor — a controlled SSL-stripping tap for diagnosing a secured channel,
or a bridge between a clear-text master and a TLS device (or the reverse).

See [TLS](tls.md) for certificates; lab certificates can be generated from any
TLS section, with `tools\New-ScopLabCerts.ps1`, or taken from the ready-made set
in `resources\certs`.

!!! warning "Use it only where you are allowed to"
    A man-in-the-middle tap — especially one that decrypts TLS — must only be
    used on systems and links you own or are explicitly authorised to inspect.
