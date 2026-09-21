# source/main/gfx/DustPool.h

> A fixed pool of reusable particle emitters for one effect (dust, clumps, sparks, drips, splashes, ripples).

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`DustPool.cpp`](DustPool.cpp.md) · [`GfxActor.cpp`](GfxActor.cpp.md) · [`GfxScene.cpp`](GfxScene.cpp.md) · [`physics/water/Buoyance.cpp`](../physics/water/Buoyance.cpp.md) · [`physics/water/ScrewProp.cpp`](../physics/water/ScrewProp.cpp.md)
**Tier floor** — T2


## Purpose

Instead of creating particle systems per contact, each effect kind owns a small pool of pre-created emitters; each frame, requests fill free slots, and `update` configures and fires exactly those. Requests beyond capacity are dropped. Implementation: [`DustPool.cpp`](DustPool.cpp.md).

## State

```text
RECORD DustPool
  size ≤ 100 slots, each: particle system (from the named template), scene node, position, velocity, colour, rate, type
  allocated : int (slots requested this frame)
  kinds: NORMAL (dust), RUBBER (tyre smoke), DRIP, VAPOUR, SPLASH, RIPPLE, SPARKS, CLUMP
```

## API

`malloc(pos, vel, colour = sandy)`, `allocClump`, `allocSmoke`, `allocSparks` (ignored below 0.1 m/s), `allocVapour(pos, vel, wet_time)`, `allocDrip(pos, vel, wet_time)`, `allocSplash`, `allocRipple`, `update`, `setVisible`, `Discard(scene)` (must be called before destruction).
