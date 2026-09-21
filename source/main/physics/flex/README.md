# source/main/physics/flex

> Things whose shape follows the soft body: skinned meshes (flexbodies), generated wheel and body meshes, and wing segments.

Most of this chapter is **graphics driven by physics**: it reads a snapshot of node positions and writes vertex buffers, never node forces. The exception is [`FlexAirfoil`](FlexAirfoil.cpp.md), which applies wing forces during the physics step and also owns the wing's mesh.

Threading: deformation (`computeFlexbody`, `flexitCompute`) runs on worker threads against the node snapshot; uploads run on the main thread.

## Reading order

1. [`Locator_t.h`](Locator_t.h.md), [`Flexable.h`](Flexable.h.md)
2. [`FlexBody.h`](FlexBody.h.md) → [`FlexBody.cpp`](FlexBody.cpp.md)
3. [`FlexFactory.h`](FlexFactory.h.md) → [`FlexFactory.cpp`](FlexFactory.cpp.md)
4. [`FlexObj.h`](FlexObj.h.md) → [`FlexObj.cpp`](FlexObj.cpp.md) — the old-style cab body
5. [`FlexMesh.h`](FlexMesh.h.md) → [`FlexMesh.cpp`](FlexMesh.cpp.md), [`FlexMeshWheel.h`](FlexMeshWheel.h.md) → [`FlexMeshWheel.cpp`](FlexMeshWheel.cpp.md)
6. [`FlexAirfoil.h`](FlexAirfoil.h.md) → [`FlexAirfoil.cpp`](FlexAirfoil.cpp.md)
