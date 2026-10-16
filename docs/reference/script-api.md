# Script API

The functions and values a [RTU script](../shared/scripting.md) can use. Lua
5.4's `string`, `table`, `math` and `utf8` libraries are available; `io`, `os`,
`package`, `require` and `load` are not.

## Values

| Point kind | Lua value |
|---|---|
| binary (single point, coil, discrete input) | `true` / `false` |
| double-bit | `DPI.INTERMEDIATE` (0), `DPI.OPEN` (1), `DPI.CLOSED` (2), `DPI.INDETERMINATE` (3) |
| analog, setpoint, register | number |
| counter | non-negative integer |
| octet string | string |

## Reading and writing points

| Function | |
|---|---|
| `get(tag)` | the point's current value |
| `set(tag, value [, quality])` | write through the protocol's update path — an event when the point changes |
| `add(tag, delta)` | counters / accumulators: value + delta (wraps at the point's rollover) |
| `quality(tag)` | the point's quality flags (protocol bits) |
| `quality(tag, flags)` | set the quality flags, value unchanged |
| `tags()` | array of the RTU's tag names |
| `is_tag(name)` | true when the name is a tag of this RTU |

## Handlers and timers

| Function | Called when |
|---|---|
| `on_command(tag, fn(cmd))` | a binary control addresses the tag's point |
| `on_setpoint(tag, fn(sp))` | a setpoint / analog output addresses it |
| `on_change(tag, fn(value, old))` | the point's value changed (any writer) |
| `on_event(name, fn(info))` | `start`, `stop`, `master_online`, `master_offline`, `time_sync`, `counter_freeze`, `restart`, `sim_on`, `sim_off` |
| `every_ms(ms, fn)` | periodically (first call after `ms`); returns a timer id |
| `after_ms(ms, fn)` | once; returns a timer id |
| `cancel(id)` | stops a timer |
| `wait_ms(ms)` | inside a handler or timer: suspend, resume later |

## Decisions

```lua
return ACCEPT                 -- or return nothing
return REJECT(code)           -- "NOT_SUPPORTED", "ALREADY_ACTIVE", "HARDWARE_ERROR", "LOCAL",
                              -- "NOT_AUTHORIZED", "TIMEOUT", "FORMAT_ERROR"; Modbus also
                              -- "ILLEGAL_VALUE", "BUSY" or an exception number
return ACCEPT, 1500           -- answer the master 1500 ms later
return REJECT(code), 300      -- a late refusal (0 or less = no delay)
```

## The `cmd` table (`on_command`)

| Field | Meaning |
|---|---|
| `tag`, `index` | the target point |
| `state` | IEC / Modbus: `"ON"`, `"OFF"`, `"HIGHER"`, `"LOWER"` |
| `op` | `"LATCH_ON"`, `"LATCH_OFF"`, `"PULSE_ON"`, `"PULSE_OFF"`, `"NUL"` (IEC: from the QU qualifier) |
| `tcc` | DNP3: `"TRIP"`, `"CLOSE"`, `"NUL"` |
| `phase` | `"SELECT"`, `"OPERATE"` / `"EXECUTE"`, `"DIRECT"`, `"DIRECT_NOACK"` |
| `count`, `on_ms`, `off_ms` | DNP3 CROB count and times |
| `queue`, `clear` | DNP3 CROB flag bits |
| `master`, `master_ip` | the master's link address / IP address when known |
| `raw` | protocol detail — DNP3 `code`, `func`, `group`; IEC `type_id`, `qu`, `value`; Modbus `fc`, `address`, `qty`, `bit` |

## The `sp` table (`on_setpoint`)

`tag`, `index`, `value` (engineering value as the point shows it), `phase`,
`master`, and `raw` — DNP3 `var` (1 int32, 2 int16, 3 float, 4 double), `func`,
`group` = 41; IEC `type_id` and the wire value; Modbus `fc`, `address`, `qty`.

## Other functions

| Function | |
|---|---|
| `now_ms()` | station time, ms since 1970 (follows time synchronisation) |
| `mono_ms()` | monotonic ms, for intervals |
| `print(...)`, `log(...)` | the Script console and Status Messages |
| `random()`, `random(a, b)` | random numbers |
| `bit(v, n)`, `bits(v, a, b)`, `with_bit(v, n, on)` | bit helpers for register words |
| `word_lo(v)`, `word_hi(v)`, `to_int16(v)`, `to_int32(v)` | word helpers |
| `help()` | prints this API in the console |

## Standard library

### Breaker

```lua
local cb = Breaker.new{ status = "CB1_ST", travel_ms = 80, stuck = false }
on_command("CB1_CMD", cb:handler())   -- SELECT accepted (refused when stuck), then trip / close
```

`status` is a double-bit or binary tag (detected from its value). `:trip()` and
`:close()` write *intermediate*, wait `travel_ms`, then write the end position
and count the operation (`cb.ops`); `:is_closed()` reads the status tag live.
`Breaker.wants_open(cmd)` tells whether a command means open (`true`), close
(`false`) or neither (`nil`) for any protocol.

### Bay

```lua
local bay = Bay.new{ Un_kV = 138, Pmax_MW = 50, pf = 0.95, f_Hz = 60, load = 0.6,
                     unbalance = 0.02, noise = 0.005 }
bay.closed = cb:is_closed(); bay:step(1)
set("P_MW", bay.P_MW)
```

Settings: `Un_kV`, `Pmax_MW`, `pf` (`pf_lead` for a capacitive load), `f_Hz`,
`load` (0..1), `f_droop`, `f_walk`, `f_dev`, `unbalance`, `noise`, `du`
(relative voltage deviation), `closed`. Outputs after `:step(dt_s)`: `P_MW`,
`Q_MVAr`, `S_MVA`, `Ia_A` `Ib_A` `Ic_A`, `Iab_A` `Ibc_A` `Ica_A`, `Ua_kV` `Ub_kV`
`Uc_kV`, `Uab_kV` `Ubc_kV` `Uca_kV`, `f_Hz`, `E_kWh`. An open breaker zeroes
currents and powers; voltages stay.

### TapChanger

```lua
local tap = TapChanger.new{ min = 1, max = 17, pos = 9, delay_ms = 2000, step_pct = 1.25,
                            position = "TAP_POS" }
```

`:raise()` / `:lower()` move one step after `delay_ms` (refused at the limits or
while moving), `:move_to(n)` steps until `n` — a new target during a move just
retargets it — and `:ratio()` gives the voltage ratio around `neutral`. With
`position` the position tag is written after each step.

### Signal

`Signal.ramp{ min, max, rate, bounce }` (units per second; `bounce` reverses at the ends instead of wrapping), `Signal.sine{ min, max, period_s }`,
`Signal.walk{ min, max, delta, value }`, `Signal.lag{ tau_s }` (first-order
filter: `:step(dt, input)`). Each `:step(dt_s)` returns the next value.

### LoadProfile

```lua
local day = LoadProfile.hourly{ [0]=0.4, [6]=0.5, [9]=0.8, [18]=0.9, [23]=0.5 }
bay.load = day:at(now_ms())   -- interpolated within the hour
```

An hour that is not given repeats the previous one; the factor is interpolated
between one hour and the next.
