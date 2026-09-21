# source/main/gameplay/TyrePressure.h

> Adjustable tyre pressure: rescales the spring constant of all wheels2 tyre beams.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`TyrePressure.cpp`](TyrePressure.cpp.md) · [`physics/Actor.h`](../physics/Actor.h.md)
**Tier floor** — T2


## Purpose

`wheels2` tyre beams register themselves at spawn; the player can inflate/deflate at runtime. Implementation: [`TyrePressure.cpp`](TyrePressure.cpp.md).

## State

```text
RECORD TyrePressure
  actor; beams : list<beam index>; pressure : 0..100 = 50 (psi-like units)
  pressing : bool; pressed_timer : s          # keeps the gauge visible 1.5 s after release
```

## API

`AddBeam(i)`, `IsEnabled()` (any beams), `UpdateInputEvents(dt)`, `ModifyTyrePressure(Δ) → changed?`, `GetCurPressure`, `IsPressurizing`.
