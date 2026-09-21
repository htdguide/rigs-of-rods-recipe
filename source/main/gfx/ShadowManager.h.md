# source/main/gfx/ShadowManager.h

> Shadow technique configuration: parallel-split shadow maps from the main light.

**Needs** — [`Application.h`](../Application.h.md) · [`terrain/OgreTerrainPSSMMaterialGenerator.h`](../terrain/OgreTerrainPSSMMaterialGenerator.h.md)
**Used by** — [`ShadowManager.cpp`](ShadowManager.cpp.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md) · [`terrain/TerrainGeometryManager.cpp`](../terrain/TerrainGeometryManager.cpp.md)
**Tier floor** — T2


## Purpose

One per terrain. Configures the renderer's shadows from `gfx_shadow_type` and `gfx_shadow_quality`, and tells the terrain material generator how to receive them. Implementation: [`ShadowManager.cpp`](ShadowManager.cpp.md).

## State

```text
RECORD ShadowManager = { pssm setup?; depth shadows = false; texture count = 3; quality 0..3; lambda }
```

## API

`loadConfiguration`, `updatePSSM` (no-op), `updateTerrainMaterial(profile)`.
