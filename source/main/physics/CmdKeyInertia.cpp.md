# source/main/physics/CmdKeyInertia.cpp

> Loading inertia_models.cfg and the curve-shaped rate limiters.

**Needs** — [`CmdKeyInertia.h`](CmdKeyInertia.h.md) · [`Application.h`](../Application.h.md) · [`utils/Utils.h`](../utils/Utils.h.md)
**Used by** — callers of [`CmdKeyInertia.h`](CmdKeyInertia.h.md) (see its Used by)
**Tier floor** — T2

## Purpose

Turns a step input (key pressed / released) into a smoothed output whose shape follows a named curve.

## State

See [`CmdKeyInertia.h`](CmdKeyInertia.h.md).

## `LoadDefaultInertiaModels`

**Contract** — reads `inertia_models.cfg` from any resource group. Format: lines trimmed; empty lines and lines starting with `;` ignored; a line with no comma starts a model named by the line; a line `x, y` appends a point to the current model's spline. Failure to open is logged, leaving no models.

**Notes** — splines are Catmull-Rom curves evaluated with a *global* parameter t ∈ [0, 1] spread evenly across segments (not by x); only the resulting y is used. A rebuild must reproduce that parameterisation, not look curves up by x.

## `SetCmdKeyDelay`

**Contract** — stores delays only if > 0 (otherwise logs and keeps 0), looks up both curves (unknown name → logged, curve left unset). Always returns 0.

## `CalcCmdKeyDelay(input, dt)`

```text
IF either curve is unset: RETURN input                 # no inertia
rel = |input| − |last|; diff = input − last
IF |diff| < 0.002: time = 0
time += dt
step(delay, curve) = curve.interpolate(min(delay·time, 1)).y · 0.001
IF diff > 0: out = last + (rel > 0 ? step(start) : rel < 0 ? step(stop) : 0); out = min(out, input)
IF diff < 0: out = last − (rel > 0 ? step(start) : rel < 0 ? step(stop) : 0); out = max(out, input)
last = out; RETURN out
```

"Start" applies when moving away from zero, "stop" when moving toward zero; the output never overshoots the input. The 0.001 scale reflects per-step increments at 2 kHz.

## `SimpleInertia.CalcSimpleDelay(input, dt)`

```text
IF input: spline_time = min(1, spline_time + dt/start_delay)
ELSE:     spline_time = max(0, spline_time − dt/stop_delay)
RETURN input ? (start curve ? start.y(spline_time) : 1) : (stop curve ? stop.y(spline_time) : 0)
```
