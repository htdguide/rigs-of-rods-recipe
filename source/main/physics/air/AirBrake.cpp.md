# source/main/physics/air/AirBrake.cpp

> Airbrake drag force and its simple quad mesh.

**Needs** — [`AirBrake.h`](AirBrake.h.md) · [`Application.h`](../../Application.h.md) · [`SimData.h`](../SimData.h.md) · [`Actor.h`](../Actor.h.md) · [`gfx/GfxActor.h`](../../gfx/GfxActor.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md)
**Used by** — callers of [`AirBrake.h`](AirBrake.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Adds drag proportional to deployment. The visual quad (two-sided, 4 vertices, texture coordinates from the file) is built here but animated by the graphics actor.

## State

See [`AirBrake.h`](AirBrake.h.md).

## `applyForce`

```text
ρ = air_density(ref.y)                       # troposphere: p = 101325·(1 − 0.0065·h/288.15)^5.24947; ρ = p·1.20896e-5
wind = −ref.velocity; s = |wind|
drag = 1.2 · area · sin(|ratio · max_angle| in radians, using 57.3°/rad) · 0.5 · ρ · s / 4 · wind
ref, x, y, a each receive + drag                 # the same force on all four nodes
```

## `updatePosition(ratio)`

Stores the deployment ratio (the actor sets it from `airbrake_intensity / 5`).

**Notes** — the air-density model (ISA troposphere, valid to 11 km) is repeated in several aero files; a rebuild should share it.
