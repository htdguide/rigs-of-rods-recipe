# source/main/physics/flex/FlexFactory.cpp

> Flexbody/mesh-wheel creation and the binary flexbody cache file format.

**Needs** — [`FlexFactory.h`](FlexFactory.h.md) · [`Application.h`](../../Application.h.md) · [`Actor.h`](../Actor.h.md) · [`resources/CacheSystem.h`](../../resources/CacheSystem.h.md) · [`FlexBody.h`](FlexBody.h.md) · [`FlexMeshWheel.h`](FlexMeshWheel.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md) · [`utils/PlatformUtils.h`](../../utils/PlatformUtils.h.md) · [`resources/rig_def_fileformat/RigDef_File.h`](../../resources/rig_def_fileformat/RigDef_File.h.md) · [`ActorSpawner.h`](../ActorSpawner.h.md)
**Used by** — callers of [`FlexFactory.h`](FlexFactory.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Mesh loading and naming for flex visuals; binding cache persistence.

## State

See [`FlexFactory.h`](FlexFactory.h.md).

## `CreateFlexBody`

Load the mesh, **clone it per actor** (each flexbody deforms its own copy) under a unique name, create an entity with per-actor materials, take the next cached record if a cache was loaded, rotation = Z·Y·X from degrees, construct the [`FlexBody`](FlexBody.cpp.md), queue it for saving if caching is enabled, set id and original mesh name.

## `CreateFlexMeshWheel`

Rim entity from the rim mesh (per-actor materials), a scene node under the actor's wheel group, a [`FlexMeshWheel`](FlexMeshWheel.cpp.md), then the tyre entity from the generated mesh.

## Cache file

```text
path: <cache dir>/flexbodies_mod_<n>.dat
"RoR FlexBody\0"  { u32 format_version = 1, u32 count }
per flexbody: header (raw struct) · locators[count] · dst_pos[count] · src_normals[count] · colours[count] if HAS_TEXTURE_BLEND
(faulty records carry only the header)
```

Load succeeds only when signature and version match. The cache is loaded before processing and saved after, only if it was not loaded.

**Notes** — the cache entry number is never assigned (stays −1), so opening the file always fails and the cache is effectively **disabled** in this snapshot, even with `gfx_flexbody_cache` on. The format writes raw in-memory structs (host byte order and padding); a rebuild that revives it should define a portable layout rather than copy this one.
