# source/main/terrain/TerrainGeometryManager.cpp

> OTC loading, terrain import/cache, blend maps, and the triangle-exact height query.

**Needs** — [`TerrainGeometryManager.h`](TerrainGeometryManager.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`Application.h`](../Application.h.md) · [`resources/ContentManager.h`](../resources/ContentManager.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`gui/panels/GUI_LoadingWindow.h`](../gui/panels/GUI_LoadingWindow.h.md) · [`Terrain.h`](Terrain.h.md) · [`resources/terrn2_fileformat/Terrn2FileFormat.h`](../resources/terrn2_fileformat/Terrn2FileFormat.h.md) · [`gfx/ShadowManager.h`](../gfx/ShadowManager.h.md) · [`OgreTerrainPSSMMaterialGenerator.h`](OgreTerrainPSSMMaterialGenerator.h.md) · [`resources/otc_fileformat/OTCFileFormat.h`](../resources/otc_fileformat/OTCFileFormat.h.md)
**Used by** — callers of [`TerrainGeometryManager.h`](TerrainGeometryManager.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`TerrainGeometryManager.h`](TerrainGeometryManager.h.md).

## State

See header.

## `InitTerrain(otc)`

```text
parse the master .otc, then each page's .otc (empty page file accepted; parse exceptions are reported but loading continues for legacy maps)
create the renderer's terrain group: X-Z aligned, page size, world size, origin; cache files "<base>_OGRE_<version>_<x>_<z>.mapbin" in the cache group
configure defaults:
  material generator = custom (terrain names a material: clone it, add the global normal map) or the PSSM generator
  pixel error, light-map direction & composite colours from the main light, import sizes (terrain size, world size, input scale = world_size_y, batch sizes)
  PSSM profile: lightmap, normal/specular mapping (forced on for OpenGL, else as configured), parallax, global colour map, depth shadows; shadow manager adjusts it
  blend-map, composite-map sizes and distance, skirt, light-map size; casts shadows if PSSM shadows; no vertex compression (Hydrax)
  layers of the first page (world size, diffuse+specular and normal+height textures)
FOR EACH page: flat → define at height 0; cached file exists (and cache enabled) → define from cache;
  else load the heightmap (".raw": raw_size², 8 or 16 bit; others: image; optional X/Y flips) → define from image, mark "new geometry"; failure → flat
load all pages synchronously
copy page (0,0)'s height data, size, world size, position; compute min/max height (renderer's values are unreliable); flat if max − min < ε
IF new geometry: without a custom material, set up layers and blend maps per page; save all pages to cache unless disabled
free temporary resources
```

**Blend maps** — for layers 1.., load the layer's blend image (resized to the blend-map size) and take its R, G, B or A channel × layer alpha as the blend weight. Optional debug: dump each blend map as a 16-bit PNG.

## `getHeightAt(x, z)`

```text
IF spec flat: RETURN 0
tx = (x − base − pos.x) / ((size − 1)·scale);  ty = (z + base − pos.z) / ((size − 1)·−scale)
IF outside (0, 1) on either axis: RETURN terrain definition's water_bottom_height
IF is_flat: RETURN min_height
RETURN height on the triangle containing (tx, ty) in the renderer's triangulation:
  grid cell from floor(t·(size − 1)); rows alternate diagonal direction (even rows split 0→2, odd rows 1→3);
  solve the plane through the three sample heights
```

The alternating diagonal matches how the renderer tessellates, so physics contact matches the visible surface exactly.

**Notes** — only page (0, 0) is sampled: on multi-page terrains other pages read as "outside" (water bottom height). Recorded as observed; RoR terrains are in practice single-page.

## `getNormalAt(x, y, z)`

Finite difference with 0.1 m step: `unit(h(x−0.1, z) − y, 0.1, y − h(x, z+0.1))`, using the node height y as the centre sample.

## Light

`UpdateMainLightPosition` pushes the light direction/diffuse and scene ambient into the terrain options and updates the group; `updateLightMap` re-derives light maps of pages not already updating.
