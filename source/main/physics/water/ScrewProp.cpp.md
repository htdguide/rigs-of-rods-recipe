# source/main/physics/water/ScrewProp.cpp

> Screw propeller thrust and rudder deflection.

**Needs** — [`ScrewProp.h`](ScrewProp.h.md) · [`Application.h`](../../Application.h.md) · [`Actor.h`](../Actor.h.md) · [`SimData.h`](../SimData.h.md) · [`ActorManager.h`](../ActorManager.h.md) · [`gfx/DustPool.h`](../../gfx/DustPool.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md) · [`audio/SoundScriptManager.h`](../../audio/SoundScriptManager.h.md) · [`terrain/Terrain.h`](../../terrain/Terrain.h.md) · [`gfx/GfxWater.h`](../../gfx/GfxWater.h.md)
**Used by** — callers of [`ScrewProp.h`](ScrewProp.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

A single force vector — no hydrodynamics beyond "is the prop under water".

## State

See [`ScrewProp.h`](ScrewProp.h.md).

## `updateForces(update)`

```text
IF no water: RETURN
depth = wave_height(ref) − ref.y; IF depth < 0: RETURN              # prop out of water
dir = unit(back − ref); IF reverse: dir = −dir
F = throttle·full_power · rotate(dir, about unit(ref − up), rudder degrees)
ref.forces += F
IF update AND throttle > 0.1: splash (strength 10 if depth < 0.2 else 5) and ripple along F/full_power
```

## Controls

`setThrottle(x)`: clamp to −1..1; throttle = |x|, reverse = x < 0; engine sound pitch = (0.5 + |x|/2)·100. `setRudder(x)`: clamp, store 45·x. `toggleReverse` zeroes throttle. `reset` = throttle 0, rudder 0, forward.

**Notes** — "HP" in the file is applied as newtons with no conversion; boat content is tuned against that.
