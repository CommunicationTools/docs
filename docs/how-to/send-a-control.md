# Send a control

Operate an output on a device from a Master tool.

1. Make sure the channel is **started** and the station is online.
2. Open the station's **Controls** tab in the point map.
3. Choose the control point and the operation:
    - **Direct-Operate** — send the command immediately.
    - **Select-before-Operate (SBO)** — select, then operate, matching devices
      that require it.
4. Send, and read the result in the status log and the
   [Communication Monitor](../shared/monitor.md) (the command's confirmation and
   status code).

Protocol specifics:

- **DNP3** — CROB (g12) for binary outputs, Analog Output (g41) for analog;
  control status codes are shown on the result.
- **IEC 101 / 104** — single / double / regulating commands and setpoints. The
  command mode is **Direct execute** or **Select then execute**. With select
  then execute, the master sends the SELECT and waits for its positive
  confirmation (ACTCON) before it sends the EXECUTE. If the device denies the
  selection, or no confirmation arrives in time, the execute is cancelled; the
  status log says which ("selection confirmed", "selection denied" or
  "selection timed out").
- **Modbus** — write single / multiple coils or registers.
