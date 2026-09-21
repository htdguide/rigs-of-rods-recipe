# source/main/gameplay/TorqueCurve.h

> Normalised engine torque as a function of rpm: named models from torque_models.cfg or a custom curve from the truck file.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`Engine.cpp`](Engine.cpp.md) · [`TorqueCurve.cpp`](TorqueCurve.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md)
**Tier floor** — T2


## Purpose

Each [`Engine`](Engine.h.md) owns one; the engine multiplies its peak torque by this curve. Implementation: [`TorqueCurve.cpp`](TorqueCurve.cpp.md).

## State

```text
RECORD TorqueCurve
  splines    : map<name, Spline>       # Catmull-Rom through (rpm, fraction) points
  used       : Spline?; used_model : name     # starts as "default"
  CUSTOM = "CustomModel"
```

## API

`getEngineTorque(rpm)`, `setTorqueModel(name) → 0 | 1 (unknown, keeps current)`, `CreateNewCurve(name = CUSTOM) → created?`, `AddCurveSample(rpm, fraction, model = CUSTOM)`, `spaceCurveEvenly(spline) → 0 ok | 1 not ascending | 2 none`, `getUsedSpline`, `getTorqueModel`.
