# source/main/terrain/TerrainGeometryManager.h

> Heightfield terrain built from .otc page configs via the renderer’s terrain component, with its own fast height lookup.

**Needs** — [`Application.h`](../Application.h.md) · [`utils/ConfigFile.h`](../utils/ConfigFile.h.md) · [`resources/otc_fileformat/OTCFileFormat.h`](../resources/otc_fileformat/OTCFileFormat.h.md)
**Used by** — [`gfx/GfxScene.cpp`](../gfx/GfxScene.cpp.md) · [`gfx/SkyManager.cpp`](../gfx/SkyManager.cpp.md) · [`gfx/SkyXManager.cpp`](../gfx/SkyXManager.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`Terrain.cpp`](Terrain.cpp.md) · [`TerrainGeometryManager.cpp`](TerrainGeometryManager.cpp.md) · [`TerrainObjectManager.cpp`](TerrainObjectManager.cpp.md)
**Tier floor** — T2


## Purpose

Wraps the [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine)'s paged terrain. Physics calls `getHeightAt` for every ground-contact node every step, so the height lookup is reimplemented here against the raw height array rather than going through the renderer. Implementation: [`TerrainGeometryManager.cpp`](TerrainGeometryManager.cpp.md).

## State

```text
RECORD TerrainGeometryManager
  spec : OTC document (pages, sizes, layers, options)
  terrain_group (renderer), new_geometry_generated
  height lookup copy: position, base = −world_size/2, scale = world_size/(size − 1), size, height array
  is_flat, min_height, max_height
```

## API

`InitTerrain(otc file) → ok`, `getTerrainGroup`, `getHeightAt(x, z)`, `getNormalAt(x, y, z)`, `getMaxTerrainSize() → (world_x, max_height, world_z)`, `isFlat`, `UpdateMainLightPosition`, `updateLightMap`.
