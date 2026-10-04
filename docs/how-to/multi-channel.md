# Create many channels at once

Build a large test set without clicking through the dialog dozens of times.

1. **+ Channel** and set **Count** to the number you want (up to 64).
2. In the **Addressing** block choose how each channel differs:
    - **Increment port**, or **Increment IP within mask** (masters, TCP/UDP),
    - increment the listening port (slaves),
    - increment the COM number (serial).
3. Decide whether the station address steps too, or tick **Same … on every
   channel**.
4. Check the preview range, then **OK**.

To copy an existing, fully-configured channel or RTU instead, use
**Duplicate…** / **Duplicate RTU…**. See
[Fan-out and duplicate](../shared/fanout.md).
