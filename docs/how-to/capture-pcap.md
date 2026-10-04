# Capture a PCAP

Record traffic to a file Wireshark can open.

1. Open the **[Communication Monitor](../shared/monitor.md)** for the channel
   (or the unified monitor in SCADAScop).
2. Start the capture and choose a file name.
3. Reproduce the traffic.
4. Stop the capture.
5. Open the file in [Wireshark](https://www.wireshark.org/) — each flow uses its
   protocol's well-known port, so the right dissector is applied automatically.

See [PCAP capture](../shared/pcap.md) for details, including mixed-protocol
captures in SCADAScop.
