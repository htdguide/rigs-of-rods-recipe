# source/main/physics/flex/Flexable.h

> The legacy three-phase deform interface still used by generated wheel meshes.

**Needs** — nothing in this repository (only standard or third-party headers)
**Used by** — [`FlexMesh.h`](FlexMesh.h.md)
**Tier floor** — T2


## Purpose

Deformable wheel visuals implement it so the graphics actor can split work: prepare (main thread), compute (worker thread), final (main thread, upload). Flexbodies no longer use it.

## State

Interface only.

## Interface

```text
INTERFACE Flexable
  flexitPrepare() → bool     # main thread; place scene nodes
  flexitCompute()            # worker thread; recompute vertex positions/normals from node snapshot
  flexitFinal() → Vec3       # main thread; upload vertices, return the mesh centre
  setVisible(bool)
```
