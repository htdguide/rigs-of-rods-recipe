# source/main/physics/flex/FlexMesh.h

> Generated mesh for classic wheels (wheels, wheels2): two face discs and a tread band built on the wheel nodes.

**Needs** — [`Application.h`](../../Application.h.md) · [`Flexable.h`](Flexable.h.md) · [`SimData.h`](../SimData.h.md)
**Used by** — [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`FlexMesh.cpp`](FlexMesh.cpp.md) · [`FlexMeshWheel.h`](FlexMeshWheel.h.md)
**Tier floor** — T2


## Purpose

Visual for wheels without a mesh file. Implements [`Flexable`](Flexable.h.md). Implementation: [`FlexMesh.cpp`](FlexMesh.cpp.md).

## State

```text
RECORD FlexMesh
  rays, rimmed (wheels2 has separate rim and tyre rings)
  vertex_nodes[]: [axis1, axis2, face ring (2 per ray), band ring (2 per ray), (rimmed: outer face ring 2 per ray)]
  vertices[] {position, normal, uv}; face indices, band indices; two submeshes (face material, band material)
  centre (last computed)
```
