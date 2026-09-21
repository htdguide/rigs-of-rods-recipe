# source/main/utils/MeshObject.h

> A mesh instance attached to a scene node, with automatic level-of-detail discovery.

**Needs** — [`Application.h`](../Application.h.md) · [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine)
**Used by** — [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`gfx/GfxData.h`](../gfx/GfxData.h.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md) · [`terrain/TerrainObjectManager.h`](../terrain/TerrainObjectManager.h.md) · [`MeshObject.cpp`](MeshObject.cpp.md)
**Tier floor** — T2

## Purpose

The common way RoR places a static or prop mesh (props, terrain objects, wheels' rim meshes). Wraps "load mesh, register LODs, create entity, attach" into one constructor. Implementation: [`MeshObject.cpp`](MeshObject.cpp.md).

## State

```text
RECORD MeshObject
  scene_node   : scene node (given by caller, not owned)
  entity       : optional<entity>     # absent if loading failed
  mesh         : mesh resource
  cast_shadows : bool = true
```

## `MeshObject(mesh_name, resource_group, entity_name, scene_node)`

**Contract** — loads (or reuses) the mesh, creates the entity and attaches it. On any engine error the object is left without an entity and the error is logged; callers must tolerate a missing entity.

## `setMaterialName(name)` · `setCastShadows(bool)` · `setVisible(bool)`

**Contract** — forward to the entity/node. `setVisible` hides the node if anything is attached, otherwise the entity directly.

## Getters

Entity, scene node, loaded mesh.
