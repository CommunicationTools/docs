# Simulation

The **Slave** tools can drive their own point values automatically, so an
outstation / RTU produces changing data without anyone editing values by hand —
useful for exercising a master, a chart, an alarm rule or a capture.

## Per-point simulation

Open a station's point window and give a point a **simulation** instead of a
fixed value. The pattern depends on the point kind:

- **Analog / measurand** — **Sine**, **Ramp**, **Random** or **Counter**, with a
  period, amplitude and min / max (engineering range).
- **Counter** — a steadily increasing count at the chosen rate.
- **Binary / indication** — a **toggle** at the chosen period.
- **Double-bit** — cycles through its states.

Each point keeps its own phase, so points on the same station do not move in
lock-step (random points use independent sequences). A simulation is saved with
the point in the [workspace](workspaces.md) and travels with the RTU when it is
duplicated, exported or copied.

## Start, stop and speed

Simulation runs only while it is switched on:

- In a standalone Slave tool, start / stop simulation and set its **speed** from
  the tool.
- In **SCADAScop**, one control drives the point simulation of **every** slave
  channel at once (DNP3, IEC 104, IEC 101). The **speed** runs from **0.1× to
  10×** of real time; the state and speed are shown in the status bar and saved
  in the `.ssw`.

Speeding time up is the quick way to watch a ramp fill, a counter roll over or a
chart draw a full curve in seconds.

!!! tip "Simulation also runs headless"
    Simulation works with no window at all. See
    [Headless and the web interface](headless.md) — the `--sim [speed]` switch
    runs the slave simulation on a server or bench rig.
