# source/main/gameplay/TyrePressure.cpp

> Pressure input handling and the pressure → stiffness law.

**Needs** — [`TyrePressure.h`](TyrePressure.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md)
**Used by** — callers of [`TyrePressure.h`](TyrePressure.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Deliberately simple: pressure only changes stiffness.

## State

See [`TyrePressure.h`](TyrePressure.h.md).

## `ModifyTyrePressure(Δ)`

`p' = clamp(p + Δ, 0, 100)`; unchanged → false. Otherwise every registered beam gets `k = 10000 + p'·10000` and p = p'. Called with Δ = 0 at spawn to initialise stiffness.

## `UpdateInputEvents(dt)`

Holding "less": Δ = `p·(1 − 2^(dt/2))` clamped to [−10·dt, −dt]; "more": `p·(2^(dt/2) − 1)` clamped to [dt, 10·dt] — exponential change with a floor and ceiling rate; air sound while it changes; on release stop sound and show the gauge for 1.5 s.
