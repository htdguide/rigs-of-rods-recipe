# source/main/physics/water/Buoyance.h

> Hull buoyancy and hydrodynamic drag computed from submerged cab triangles.

**Needs** — [`Application.h`](../../Application.h.md) · [`utils/Vec3.h`](../../utils/Vec3.h.md)
**Used by** — [`gfx/GfxActor.cpp`](../../gfx/GfxActor.cpp.md) · [`physics/Actor.cpp`](../Actor.cpp.md) · [`physics/ActorForcesEuler.cpp`](../ActorForcesEuler.cpp.md) · [`physics/ActorManager.cpp`](../ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`physics/Savegame.cpp`](../Savegame.cpp.md) · [`Buoyance.cpp`](Buoyance.cpp.md)
**Tier floor** — T2


## Purpose

An actor with buoyant cabs owns one `Buoyance`. Each step the buoyant cab triangles are evaluated against the (possibly wavy) water surface. Because the computation is expensive and reads the water model, positions are copied into a node cache first so it can run off the node array. Implementation: [`Buoyance.cpp`](Buoyance.cpp.md).

## State

```text
RECORD BuoyCachedNode = { position, velocity, forces : Vec3; node_num }
RECORD BuoyDebugSubCab = { a, b, c, normal, drag : Vec3; volume }     # for the debug view

RECORD Buoyance
  cached_nodes    : list<BuoyCachedNode>      # each buoyant cab vertex once
  projected_nodes : list<BuoyCachedNode>      # the same, advanced in time for the async variant
  sink            : bool                      # "sink" toggled by scripts/savegame: drag only, no lift
  update          : bool                      # this step may emit particles / debug data
  debug_view, debug_subcabs, total_steps, last_sample_steps
  splash, ripple  : particle pools
ENUM type = NORMAL | DRAGONLY | DRAGLESS
```

## API

`cacheBuoycabNode(node) → id` (reuses an existing entry for the same node), `computeNodeForce(a, b, c, type, timeshift)`.
