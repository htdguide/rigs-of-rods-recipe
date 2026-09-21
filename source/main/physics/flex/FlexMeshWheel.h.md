# source/main/physics/flex/FlexMeshWheel.h

> Mesh wheels (meshwheels, meshwheels2, flexbodywheels): a rigid rim mesh plus a generated deforming tyre.

**Needs** — [`ForwardDeclarations.h`](../../ForwardDeclarations.h.md) · [`FlexMesh.h`](FlexMesh.h.md)
**Used by** — [`gfx/GfxActor.cpp`](../../gfx/GfxActor.cpp.md) · [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`FlexFactory.cpp`](FlexFactory.cpp.md) · [`FlexMeshWheel.cpp`](FlexMeshWheel.cpp.md)
**Tier floor** — T2


## Purpose

Implements [`Flexable`](Flexable.h.md). Created only by [`FlexFactory`](FlexFactory.cpp.md). For flexbody wheels the generated tyre is given a transparent material and a real flexbody provides the tyre visual instead.

## State

```text
RECORD FlexMeshWheel
  rim entity + scene node (owned), tyre entity (owned; its scene node belongs to the wheel visual)
  axis nodes 0/1, first tread node, rays, rim_radius, rim_reverse (right-side wheels)
  tyre mesh: 6·(rays + 1) vertices (a closing seam copy), 10 triangles per ray
```
