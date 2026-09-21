# source/main/physics/air/AirBrake.h

> A deployable drag panel defined by four nodes.

**Needs** — [`Application.h`](../../Application.h.md)
**Used by** — [`gfx/GfxActor.cpp`](../../gfx/GfxActor.cpp.md) · [`physics/Actor.cpp`](../Actor.cpp.md) · [`physics/ActorForcesEuler.cpp`](../ActorForcesEuler.cpp.md) · [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`AirBrake.cpp`](AirBrake.cpp.md)
**Tier floor** — T2


## Purpose

`airbrakes` in the truck format. Physics in [`AirBrake.cpp`](AirBrake.cpp.md); its mesh and scene node are handed to the graphics actor right after construction.

## State

```text
RECORD Airbrake
  ref, x, y, a : node                 # the panel's reference, axis and additional nodes
  offset       : Vec3                 # visual offset
  ratio        : 0..1                 # current deployment
  max_angle    : degrees
  area         : m²  = width × length × lift_coefficient
  mesh, entity, scene node            # visual (moved to the graphics actor at spawn)
```

## API

`Airbrake(actor, name, index, ref, x, y, a, offset, width, length, max_angle, material, tex coords, lift_coef)`, `updatePosition(ratio)`, `applyForce()`, `getRatio`, `getMaxAngle`.
