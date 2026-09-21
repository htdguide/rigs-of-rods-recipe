# source/main/physics/collision

> How nodes touch the world and each other.

Two mechanisms, deliberately different:

- **Static world** — [`Collisions`](Collisions.h.md): terrain heightfield, collision boxes from placed objects, collision triangles from meshes, indexed in a 2 m spatial hash. Each node is tested individually every step.
- **Moving bodies** — [`DynamicCollisions`](DynamicCollisions.cpp.md): each actor's *collision cabs* (triangles of its own nodes) are tested against candidate nodes found by a lazily built kd-tree ([`PointColDetector`](PointColDetector.h.md)); geometry via [`Triangle`](Triangle.h.md) and [`CartesianToTriangleTransform`](CartesianToTriangleTransform.h.md).

Both apply the same contact law, `primitiveCollision` (static + Stribeck friction, optional fluid layer), parameterised by a **ground model** from `ground_models.cfg`.

## Reading order

1. [`Triangle.h`](Triangle.h.md)
2. [`CartesianToTriangleTransform.h`](CartesianToTriangleTransform.h.md)
3. [`Collisions.h`](Collisions.h.md) → [`Collisions.cpp`](Collisions.cpp.md)
4. [`PointColDetector.h`](PointColDetector.h.md) → [`PointColDetector.cpp`](PointColDetector.cpp.md)
5. [`DynamicCollisions.h`](DynamicCollisions.h.md) → [`DynamicCollisions.cpp`](DynamicCollisions.cpp.md)
