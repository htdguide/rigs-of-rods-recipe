# source/main/physics/collision/DynamicCollisions.h

> Declares the two cab-triangle collision passes: against other actors and against the actor itself.

**Needs** — [`ForwardDeclarations.h`](../../ForwardDeclarations.h.md) · [`SimData.h`](../SimData.h.md)
**Used by** — [`physics/Actor.cpp`](../Actor.cpp.md) · [`physics/ActorManager.cpp`](../ActorManager.cpp.md) · [`DynamicCollisions.cpp`](DynamicCollisions.cpp.md)
**Tier floor** — T2


## Purpose

Free functions over raw actor arrays (cabs, collision cabs, skip counters, nodes) so the step can call them without an actor handle. Implementation in [`DynamicCollisions.cpp`](DynamicCollisions.cpp.md).

## State

Stateless.

## `ResolveInterActorCollisions(dt, detector, n_collcabs, collcabs, cabs, rates, nodes, range, submesh_gm)`

Contract — see implementation twin.

## `ResolveIntraActorCollisions(…)`

Same parameters with the intra detector and intra skip counters.
