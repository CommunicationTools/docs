# Script library

Every [RTU script](scripting.md) starts with the **standard library** already
loaded: a handful of Lua objects that model the usual pieces of a substation,
plus bit and word helpers. They are plain Lua — open **Library** in the Script
window to read the source, or copy an object into your own script and change
it. `help()` prints a one-screen summary in the console.

The objects do not know anything about protocols. They read and write points
through `get()` / `set()` by tag, return `ACCEPT` / `REJECT` like your own
handlers, and use `wait_ms()` for anything that takes time — so a method that
waits (`Breaker:trip()`, `TapChanger:raise()`) must be called from a handler or
a timer, never from the main chunk of the script.

## Breaker

A circuit breaker with travel time. Its position lives on a **status tag** — a
binary point (`true` = closed) or a double-bit point (`DPI.OPEN` / `DPI.CLOSED`,
detected from the point's value).

```lua
local cb = Breaker.new{ status = "CB1_ST", travel_ms = 120 }
on_command("CB1_CMD", cb:handler())
```

| Setting | Default | |
|---|---|---|
| `status` | — | tag of the status point (binary or double-bit); without it the breaker only keeps an internal state |
| `travel_ms` | 80 | time spent in *intermediate* between the command and the end position |
| `stuck` | `false` | `true` refuses every command with `HARDWARE_ERROR` — a jammed mechanism |
| `double_bit` | auto | force the status to be treated as double-bit (`true`) or binary (`false`) |
| `closed` | `true` | the initial state when there is no status tag |

| Method | |
|---|---|
| `cb:handler()` | a ready-made `on_command` handler: accepts the SELECT (refuses it when stuck), then trips or closes according to the command — DNP3 `TRIP` / `CLOSE`, IEC `OFF` / `ON`, latch / pulse on / off. Anything else is refused with `NOT_SUPPORTED`. |
| `cb:trip()`, `cb:close()` | move the breaker yourself: writes *intermediate*, waits `travel_ms`, writes the end position, counts the operation. Return `ACCEPT` / `REJECT`, so a handler can `return cb:trip()`. A command that asks for the position the breaker already has is accepted without moving. |
| `cb:is_closed()` | the live position, read from the status tag (the master, the simulation or a display may have moved it) |
| `cb.ops` | number of operations since the script started |
| `Breaker.wants_open(cmd)` | `true` for an open / trip command, `false` for close, `nil` for neither — whatever the protocol |

**A breaker that trips on a fault and refuses to close while the fault is on**

```lua
local cb = Breaker.new{ status = "CB1_ST", travel_ms = 120 }

on_command("CB1_CMD", function(cmd)
    if cmd.phase == "SELECT" then return ACCEPT end
    if Breaker.wants_open(cmd) == false and get("PROT_TRIP") then
        return REJECT("LOCAL")          -- interlock: relay still picked up
    end
    return cb:handler()(cmd)            -- otherwise the standard behaviour
end)

on_change("PROT_TRIP", function(tripped)
    if tripped and cb:is_closed() then cb:trip() end
end)
```

## Bay

Steady-state electrical quantities of one feeder or transformer bay from a few
settings. Call `:step(dt_s)` from a timer, then read the output fields and
publish the ones you have points for.

```lua
local bay = Bay.new{ Un_kV = 138, Pmax_MW = 50, pf = 0.95, f_Hz = 60, load = 0.6,
                     unbalance = 0.02, noise = 0.005 }

every_ms(1000, function()
    bay.closed = cb:is_closed()
    bay:step(1)
    set("P_MW",   bay.P_MW)
    set("Q_MVAR", bay.Q_MVAr)
    set("IA_A",   bay.Ia_A); set("IB_A", bay.Ib_A); set("IC_A", bay.Ic_A)
    set("UAB_KV", bay.Uab_kV)
    set("F_HZ",   bay.f_Hz)
end)
```

| Setting | Default | |
|---|---|---|
| `Un_kV` | 13.8 | nominal line-to-line voltage |
| `Pmax_MW` | 10 | rating; `load` is a fraction of it |
| `pf`, `pf_lead` | 0.95, `false` | power factor; `pf_lead = true` for a capacitive load (Q negative) |
| `f_Hz` | 60 | nominal frequency |
| `load` | 0.7 | 0..1 (up to 2 for an overload test) |
| `f_droop`, `f_ref` | 0, 0.5 | frequency droop: `f = f_Hz - f_droop * (load - f_ref)` while the breaker is closed |
| `f_walk`, `f_band` | 0, 0.2 | a random walk of the frequency, `f_walk` Hz per step, kept within ±`f_band` |
| `f_dev` | 0 | a fixed frequency offset you set yourself (an island-mode scenario) |
| `unbalance` | 0.02 | relative current unbalance between phases |
| `noise` | 0.005 | relative noise added to every output |
| `du` | 0 | relative voltage deviation (0.02 = +2 %) |
| `closed` | `true` | breaker state: open → zero currents and powers, voltages stay |

Outputs after `:step(dt_s)`: `P_MW`, `Q_MVAr`, `S_MVA`, `Ia_A` `Ib_A` `Ic_A`
(phase currents), `Iab_A` `Ibc_A` `Ica_A` (delta-branch currents), `Ua_kV`
`Ub_kV` `Uc_kV` (phase-neutral), `Uab_kV` `Ubc_kV` `Uca_kV`, `f_Hz`, and
`E_kWh` — the energy accumulated over the `dt` of every step, which is what an
accumulator point wants (`add("KWH", bay.P_MW * 1000 * dt / 3600)` publishes
the increment of one step).

Change a setting at any time between steps: `bay.load = 0.9`, `bay.du = -0.03`,
`bay.f_dev = -0.2`.

## TapChanger

An on-load tap changer with mechanical delay.

```lua
local tap = TapChanger.new{ min = 1, max = 17, pos = 9, delay_ms = 1500,
                            step_pct = 1.25, position = "TAP_POS" }

on_command("TAP_RL", function(cmd)                 -- a raise / lower command
    if cmd.phase == "SELECT" then return ACCEPT end
    if cmd.state == "HIGHER" then return tap:raise() end
    if cmd.state == "LOWER"  then return tap:lower() end
    return REJECT("NOT_SUPPORTED")
end)

on_setpoint("TAP_SP", function(sp)                 -- or an absolute setpoint
    if sp.phase == "SELECT" then return ACCEPT end
    return tap:move_to(sp.value)
end)
```

| Setting | Default | |
|---|---|---|
| `min`, `max` | 1, 17 | the tap range |
| `pos` | middle | the starting position |
| `neutral` | middle | the position where `:ratio()` is 1.0 |
| `delay_ms` | 2000 | time per step |
| `step_pct` | 1.25 | voltage change per step, in % |
| `position` | — | tag of an analog point written after every step |

| Method | |
|---|---|
| `tap:raise()`, `tap:lower()` | one step after `delay_ms`; refused with `ALREADY_ACTIVE` while a step is running and `NOT_SUPPORTED` at the end of the range |
| `tap:move_to(n)` | steps to `n`, one tap per `delay_ms`. A new target during a move just retargets it — a setpoint handler can call `move_to(sp.value)` every time without caring whether a move is in progress. |
| `tap:ratio()` | `1 + (pos - neutral) * step_pct / 100` — feed it to a bay as `bay.du = tap:ratio() - 1` so the tap moves the bus voltage |
| `tap.pos`, `tap.moving` | the current position and whether a move is running |

## Signal

Value generators. Each is a table with `:step(dt_s)` → the next value and a
`value` field with the last one.

| Generator | Settings | |
|---|---|---|
| `Signal.ramp{ min, max, rate, bounce }` | `rate` in units per second | runs from `min` to `max` and wraps, or reverses at the ends with `bounce = true` |
| `Signal.sine{ min, max, period_s }` | | a sine between `min` and `max` |
| `Signal.walk{ min, max, delta, value }` | `delta` = largest move per step | a random walk that stays within the range; `value` sets the start |
| `Signal.lag{ tau_s, value }` | | a first-order filter: `:step(dt, input)` moves `value` towards `input` with time constant `tau_s` |

```lua
local load  = Signal.walk{ min = 0.3, max = 0.9, delta = 0.01, value = 0.6 }
local temp  = Signal.sine{ min = 35, max = 65, period_s = 600 }
local tankF = Signal.lag{ tau_s = 20 }               -- the tank follows the valve slowly

every_ms(1000, function()
    bay.load = load:step(1)
    set("OIL_TEMP", temp:step(1))
    set("LEVEL_PCT", tankF:step(1, get("VALVE_PCT")))
end)
```

## LoadProfile

A load factor by hour of the day, linearly interpolated inside the hour. Hours
you do not give repeat the previous value. The hour is taken from the station
time (`now_ms()`), so it follows the [Time & Date](../protocols/scadascop.md)
setting and the simulation speed.

```lua
local day = LoadProfile.hourly{ [0] = 0.4, [6] = 0.5, [9] = 0.8, [12] = 0.7,
                                [18] = 0.95, [22] = 0.6 }
every_ms(1000, function()
    bay.load = day:at(now_ms())
    bay:step(1)
    set("P_MW", bay.P_MW)
end)
```

Run the slave simulation at 10× (the status bar control, or `scopctl sim on 10`)
and a whole day's curve takes under two and a half hours; combine it with a
[chart](charts.md) on `P_MW` to see it.

## Bit and word helpers

For register words and status words (Lua 5.4 also has the native operators
`&`, `|`, `~`, `<<`, `>>`).

| Function | |
|---|---|
| `bit(v, n)` | `true` when bit `n` (0 = LSB) of `v` is set |
| `bits(v, from, to)` | the integer in bits `from..to` of `v` |
| `with_bit(v, n, on)` | `v` with bit `n` set or cleared |
| `word_lo(v)`, `word_hi(v)` | the low / high 16 bits of a 32-bit value |
| `to_int16(v)`, `to_int32(v)` | reinterpret an unsigned word / double word as signed |

```lua
-- A Modbus status word: RUN = bit 0, ALARM = bit 1, TRIP = bit 5
local function status_word(running, alarm, trip)
    local w = 0
    w = with_bit(w, 0, running)
    w = with_bit(w, 1, alarm)
    w = with_bit(w, 5, trip)
    return w
end
set("STATUS", status_word(true, false, get("PROT_TRIP")))
print("tripped:", bit(get("STATUS"), 5))
```

## Writing your own objects

The library objects are ordinary Lua tables with a metatable. The shortest way
to a new one is to copy the closest existing object from the **Library**
viewer into your script and rename it; the pattern is always the same:

```lua
Valve = {}
Valve.__index = Valve

function Valve.new(t)
    local v = setmetatable({}, Valve)
    v.position = t.position          -- tag of the position feedback
    v.rate     = t.rate or 5         -- % per second
    v.target   = get(v.position) or 0
    return v
end

function Valve:handler()             -- a setpoint handler: move slowly to the target
    return function(sp)
        if sp.phase == "SELECT" then return ACCEPT end
        self.target = sp.value
        if self.moving then return ACCEPT end
        self.moving = true
        while math.abs(get(self.position) - self.target) > 0.01 do
            local p = get(self.position)
            local d = math.min(self.rate * 0.1, math.abs(self.target - p))
            set(self.position, p + (self.target > p and d or -d))
            wait_ms(100)
        end
        self.moving = false
        return ACCEPT
    end
end

local v = Valve.new{ position = "VALVE_PCT", rate = 10 }
on_setpoint("VALVE_SP", v:handler())
```
