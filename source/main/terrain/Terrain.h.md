# source/main/terrain/Terrain.h

> A loaded terrain: definition, managers for geometry, objects, collisions, water, sky, shadows, and the gameplay properties scripts can query.

**Needs** — [`Application.h`](../Application.h.md) · [`utils/memory/RefCountingObject.h`](../utils/memory/RefCountingObject.h.md) · [`physics/SimConstants.h`](../physics/SimConstants.h.md) · [`SurveyMapEntity.h`](SurveyMapEntity.h.md) · [`TerrainEditor.h`](TerrainEditor.h.md) · [`physics/water/Wavefield.h`](../physics/water/Wavefield.h.md)
**Used by** — [`Application.cpp`](../Application.cpp.md) · [`GameContext.cpp`](../GameContext.cpp.md) · [`GameContext.h`](../GameContext.h.md) · [`gameplay/AutoPilot.cpp`](../gameplay/AutoPilot.cpp.md) · [`gameplay/Character.cpp`](../gameplay/Character.cpp.md) · [`gameplay/Landusemap.cpp`](../gameplay/Landusemap.cpp.md) · [`gfx/DustPool.cpp`](../gfx/DustPool.cpp.md) · [`gfx/EnvironmentMap.cpp`](../gfx/EnvironmentMap.cpp.md) · [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`gfx/GfxScene.cpp`](../gfx/GfxScene.cpp.md) · [`gfx/GfxWater.cpp`](../gfx/GfxWater.cpp.md) · [`gfx/HydraxWater.cpp`](../gfx/HydraxWater.cpp.md) · [`gfx/SkyManager.cpp`](../gfx/SkyManager.cpp.md) · [`gfx/SkyXManager.cpp`](../gfx/SkyXManager.cpp.md) · [`gfx/SurveyMapTextureCreator.cpp`](../gfx/SurveyMapTextureCreator.cpp.md) · [`gfx/camera/CameraManager.cpp`](../gfx/camera/CameraManager.cpp.md) · [`gui/GUIManager.cpp`](../gui/GUIManager.cpp.md) · [`gui/OverlayWrapper.cpp`](../gui/OverlayWrapper.cpp.md) · [`gui/panels/GUI_CollisionsDebug.cpp`](../gui/panels/GUI_CollisionsDebug.cpp.md) · [`gui/panels/GUI_FlexbodyDebug.cpp`](../gui/panels/GUI_FlexbodyDebug.cpp.md) · [`gui/panels/GUI_FrictionSettings.cpp`](../gui/panels/GUI_FrictionSettings.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](../gui/panels/GUI_SurveyMap.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`main.cpp`](../main.cpp.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/ActorForcesEuler.cpp`](../physics/ActorForcesEuler.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/Savegame.cpp`](../physics/Savegame.cpp.md) · [`physics/collision/Collisions.cpp`](../physics/collision/Collisions.cpp.md) · [`physics/water/Buoyance.cpp`](../physics/water/Buoyance.cpp.md) · [`physics/water/ScrewProp.cpp`](../physics/water/ScrewProp.cpp.md) · [`physics/water/Wavefield.cpp`](../physics/water/Wavefield.cpp.md) · [`resources/CacheSystem.cpp`](../resources/CacheSystem.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`scripting/bindings/TerrainAngelscript.cpp`](../scripting/bindings/TerrainAngelscript.cpp.md) · [`system/ConsoleCmd.cpp`](../system/ConsoleCmd.cpp.md) · [`ProceduralRoad.cpp`](ProceduralRoad.cpp.md) · [`Terrain.cpp`](Terrain.cpp.md) · [`TerrainEditor.cpp`](TerrainEditor.cpp.md) · [`TerrainGeometryManager.cpp`](TerrainGeometryManager.cpp.md) · [`TerrainObjectManager.cpp`](TerrainObjectManager.cpp.md) · [`utils/MeshObject.cpp`](../utils/MeshObject.cpp.md)
**Tier floor** — T2


## Purpose

One terrain is loaded at a time, created from a [terrn2 document](../resources/terrn2_fileformat/Terrn2FileFormat.h.md) and its cache entry. It owns the per-terrain subsystems and is the script API object `TerrainClass` (method order mirrors the binding). Reference-counted. Implementation: [`Terrain.cpp`](Terrain.cpp.md).

## State

```text
RECORD Terrain
  cache_entry, definition (terrn2)
  object_manager, geometry_manager, collisions, wavefield (physics water), gfx_water (rendered water),
  shadow_manager, sky managers (Caelum / SkyX), hydrax water, terrain_editor
  main_light, sight_range = 1000 (UNLIMITED ≥ 4999), paged_detail_factor (vegetation 0 / 0.2 / 0.5 / 1),
  gravity = −9.807, disposed
```

## API

Script-visible: name, file name, resource group, GUID, version, cache entry, `isFlat`, `getHeightAt(x, z)`, spawn position/rotation, survey-map entities (add/delete/list), procedural manager. Internal: water height, managers, far clip, detail factor, gravity get/set, `GetNormalAt(x, y, z)`, max size, collision AAB, definition, telepoints, predefined actors, `initialize`, `dispose`.
