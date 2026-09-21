# source/main/gameplay/TorqueCurve.cpp

> Loads torque models and evaluates the curve with an evenly re-sampled spline.

**Needs** — [`TorqueCurve.h`](TorqueCurve.h.md) · [`Application.h`](../Application.h.md) · [`utils/Utils.h`](../utils/Utils.h.md)
**Used by** — callers of [`TorqueCurve.h`](TorqueCurve.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Splines here are parameterised by a global t ∈ [0, 1] spread evenly over their points (see [`CmdKeyInertia`](../physics/CmdKeyInertia.cpp.md) for the same caveat). To make t proportional to rpm, the curve is **re-sampled at equal rpm spacing** once after loading.

## State

See [`TorqueCurve.h`](TorqueCurve.h.md).

## Loading `torque_models.cfg`

Same line format as the inertia models: `;` comments, a bare word starts a model, `rpm, fraction` appends a point. Missing file → logged, no models (the "default" selection then fails and the curve returns 0 torque — the file ships with the game). Selecting or creating the custom model makes it active immediately.

## `getEngineTorque(rpm)`

No curve → 0; one point or zero rpm span → that point's y; else `t = clamp((rpm − first.x)/(last.x − first.x), 0, 1)`, return `spline(t).y`. Beyond the curve's ends the torque is held at the end values.

## `spaceCurveEvenly(spline)`

```text
step = smallest gap between consecutive x; IF step < 0: RETURN 1        # rpm must ascend
x = first.x; k = 1
WHILE x ≤ last.x AND k < count
  IF x > p[k].x: k += 1
  add (x, linear interpolation between p[k−1] and p[k] at x); x += step
IF the last added x < last.x AND x − last.x < 1 % of last.x: add (x, last.y)
```

The engine's spawn finaliser calls this; on failure 1 it falls back to the "default" model with the error "Points (rpm) must be in an ascending order".

**Notes** — the advance check increments `k` at most once per sample, which is fine because the sample step is the smallest gap.
