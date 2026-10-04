# Poll a device

Read values from a real or simulated device with a Master tool. The steps are
the same for every protocol; the example uses DNP3.

1. **Create a channel** in the Master (**+ Channel**), set the endpoint (serial
   port or host/port) and the station address. See
   [Channels window](../shared/channels.md).
2. **Start** the channel. The Master connects and runs its start-up sequence.
3. Open the station's **point map** to watch values arrive.
4. Open the **[Communication Monitor](../shared/monitor.md)** to see the polls
   and responses decoded.
5. Adjust the poll schedule if needed (DNP3: the Messages → Communication Flow
   tab; other tools have equivalent polling settings).

!!! tip "No hardware?"
    Run the matching Slave tool as a simulated device — see
    [First run](../getting-started/first-run.md).
