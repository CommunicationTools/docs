# Modbus

!!! note "Draft — expanding"
    This chapter is an outline. The shared windows it refers to are complete in
    [Using the tools](../shared/channels.md); DNP3 is the fully worked example.

ModbusScop is the Modbus pair: **ModbusScop Master** (client) and **ModbusScop
Slave** (server), over Modbus RTU (serial) or Modbus TCP. ModbusScop Slave can
also act as a **gateway** (MBAP ⇄ RTU).

## Addressing

A channel carries one or more **devices**, each identified by a **Unit ID**. On
serial / RTU framing, unit 0 is broadcast and 248–255 are reserved; on MBAP,
0 / 255 mean "the device itself". The tools show a note under the unit-id field.

## Points / maps

Devices expose the four Modbus tables — **Coils**, **Discrete Inputs**,
**Holding Registers**, **Input Registers** — plus a free PLC memory map in the
Slave. The register grids are resizable and have a header right-click menu
(rows per column, hide Name, show address in cell, Base 1 register numbers,
reset widths); see [Points and maps](../shared/points.md).

## Master

- Poll windows let you read/write a function code over an address range, with the
  data grid showing values live and grouping 32/64-bit values.
- Inspect / device-scan / register-scan tools help probe an unknown device.

## Slave

- Serve values from the device tables or the PLC map; edit or simulate them.
- **Gateway** mode re-frames MBAP ⇄ RTU with unit remapping and the right Modbus
  exceptions (0x0A / 0x0B) when the device side is down.

## To document

- Poll window layout and the function-code picker.
- Write single / multiple, and the confirmation flow.
- Simulation modes and the Test (no-reply / delay / forced exception) controls.
- Gateway setup end to end.
