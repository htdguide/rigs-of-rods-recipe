# source/main/terrain/Terrain.cpp

> Terrain loading sequence, sky/light/fog/water/vegetation setup, and teardown order.

**Needs** — [`Terrain.h`](Terrain.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`physics/collision/Collisions.h`](../physics/collision/Collisions.h.md) · [`resources/ContentManager.h`](../resources/ContentManager.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`gui/panels/GUI_LoadingWindow.h`](../gui/panels/GUI_LoadingWindow.h.md) · [`gui/panels/GUI_SurveyMap.h`](../gui/panels/GUI_SurveyMap.h.md) · [`gfx/HydraxWater.h`](../gfx/HydraxWater.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md) · [`gfx/ShadowManager.h`](../gfx/ShadowManager.h.md) · [`gfx/SkyManager.h`](../gfx/SkyManager.h.md) · [`gfx/SkyXManager.h`](../gfx/SkyXManager.h.md) · [`TerrainGeometryManager.h`](TerrainGeometryManager.h.md) · [`TerrainObjectManager.h`](TerrainObjectManager.h.md) · [`resources/terrn2_fileformat/Terrn2FileFormat.h`](../resources/terrn2_fileformat/Terrn2FileFormat.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [`gfx/GfxWater.h`](../gfx/GfxWater.h.md)
**Used by** — callers of [`Terrain.h`](Terrain.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

The loading order is load-bearing (collision needs geometry size; objects need collisions and water; the survey map must be rendered before actors appear).

## State

See [`Terrain.h`](Terrain.h.md).

## `initialize() → ok`

```text
gravity ← definition
10 %  object manager (ODEF handling)
14 %  shadows (load shadow configuration)
17 %  geometry manager
23 %  camera: background = ambient colour; camera at start position;
      sight range = 5000 for SkyX else gfx_sight_range; far clip = range if < 4999 (not SkyX),
      else unlimited (Hydrax: 9999·6)
25 %  sky: Caelum (terrain's config if present, else "ror_default_sky") | SkyX (terrain's .skx or SkyXDefault.skx)
      | skybox (terrain's cubemap or "tracks/skyboxcol")
27 %  light: sky's light, or a directional "MainLight" direction unit(0.785, −0.423, 0.453), ambient-coloured, shadows to 1000 m
29 %  fog (unless Caelum): none when unlimited range, else linear from 0.65·range to 0.9·range in the ambient colour
31 %  vegetation detail factor from gfx_vegetation_mode; force the compositor clear colour to black
40 %  terrain geometry from the .otc file — failure aborts loading
60 %  collisions (with max terrain size; loads ground models)
75 %  scripts: each terrain .as file (log to console suppressed meanwhile); none loaded → default terrain script; AI presets
77 %  water (unless disabled globally or by the terrain): wavefield at the water height;
      Hydrax (terrain config or default; adds a depth technique to terrain materials) or the basic water (with bottom height)
80 %  every .tobj file; land-use (traction map) if configured
90 %  terrain light/composite maps update; telepoints; dust/particle pools
92 %  survey map textures (before actors so they are not baked in)
95 %  predefined actors
set sim_terrain_name and sim_terrain_gui_name
```

## `dispose` (skipped during application shutdown for speed)

Order: sky managers → lights → Hydrax water → object manager → geometry manager → shadow manager → collisions → wavefield → unload the terrain script unit.

## Queries

`getHeightAt` and `GetNormalAt` delegate to the geometry manager; `isFlat` is false after disposal; survey-map entities live in the object manager (delete-by-id removes all with that id).
