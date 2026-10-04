# DNP3 message catalog

The messages the DNPScop Master can send (from the **Messages → Inspect** tab and
the scheduled **Communication Flow**), and the object groups involved.

## Link layer

| Message | Function |
|---|---|
| Reset Link | RESET_LINK_STATES |
| Link Status | REQUEST_LINK_STATUS |

## Application — classes

| Message | Objects |
|---|---|
| Class read | g60 (v1 = Class 0, v2 = Class 1, v3 = Class 2, v4 = Class 3) |
| Enable unsolicited | FC 20, g60 v2/v3/v4 |
| Disable unsolicited | FC 21, g60 v2/v3/v4 |

## Application — group read

Each family has a **static** and an **event** group; the Inspect row picks one,
a variation (0 = any) and a range (All / Range / Count / List).

| Family | Static | Event |
|---|---|---|
| Binary Inputs | g1 | g2 |
| Double-bit Inputs | g3 | g4 |
| Binary Outputs | g10 | g11 |
| Counters | g20 | g22 |
| Frozen Counters | g21 | g23 |
| Analog Inputs | g30 | g32 |
| Analog Outputs | g40 | g42 |
| Octet Strings | g110 | g111 |
| Binary Command Events | — | g13 |
| Analog Command Events | — | g43 |

**Read range → qualifier:** All → `0x06`; Range → `0x00` / `0x01`; Count (first
N) → `0x07` / `0x08`; List → `0x17` / `0x28`.

## Time

| Message | Objects |
|---|---|
| Write Time v1 | WRITE g50v1 |
| Write Time v3 (LAN) | RECORD_CURRENT_TIME, then WRITE g50v3 |
| Record Current Time | RECORD_CURRENT_TIME |
| Read Time | READ g50v1 |

## Assign class

| Message | Objects |
|---|---|
| Assign class | FC 22, g60 (class) + the static group, All or a range |

## Controls

| Message | Objects |
|---|---|
| Binary output (CROB) | g12v1, Direct-Operate or Select/Operate |
| Analog output | g41 (v1 32-bit, v2 16-bit, v3 float) |
