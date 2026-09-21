# source/main/physics/flex/FlexFactory.h

> Creates flexbodies and mesh wheels for the spawner; declares the (dormant) binary flexbody cache.

**Needs** — [`utils/BitFlags.h`](../../utils/BitFlags.h.md) · [`ForwardDeclarations.h`](../../ForwardDeclarations.h.md) · [`Locator_t.h`](Locator_t.h.md) · [`resources/rig_def_fileformat/RigDef_Prerequisites.h`](../../resources/rig_def_fileformat/RigDef_Prerequisites.h.md)
**Used by** — [`physics/ActorSpawner.h`](../ActorSpawner.h.md) · [`FlexBody.cpp`](FlexBody.cpp.md) · [`FlexFactory.cpp`](FlexFactory.cpp.md)
**Tier floor** — T2


## Purpose

Centralises creation so flexbodies can be served from a cache of precomputed bindings. Implementation: [`FlexFactory.cpp`](FlexFactory.cpp.md).

## State

```text
RECORD FlexBodyRecordHeader
  vertex_count, node_center, node_x, node_y : int; center_offset : Vec3; camera_mode
  shared_buf_num_verts, num_submesh_vbufs : int
  flags : IS_FAULTY | USES_SHARED_VERTEX_DATA | HAS_TEXTURE | HAS_TEXTURE_BLEND

RECORD FlexBodyCacheData = { header; dst_pos[]; src_normals[]; src_colors[]?; locators[] }

RECORD FlexBodyFileIO
  items_to_save, loaded_items, file, format version (1), cache_entry_number = −1
  SIGNATURE = "RoR FlexBody" (NUL-terminated)

RECORD FlexFactory
  spawner, cache io, cache_enabled (gfx_flexbody_cache), cache_loaded, next_cache_index
```

## API

`CreateFlexBody(id, ref, x, y, offset, rotation°, forset nodes, forverts, mesh, group)`, `CreateFlexMeshWheel(index, axis1, axis2, first node, rays, rim_radius, rim_reverse, rim mesh, group, tyre material, group)`, `CheckAndLoadFlexbodyCache`, `SaveFlexbodiesToCache`.
