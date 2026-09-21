# source/main/gfx/GfxScene.h

> The visual scene: owns the scene manager, particle pools, the list of actor and character visuals, free-beam visuals, and the game-context sim buffer.

**Needs** — [`camera/CameraManager.h`](camera/CameraManager.h.md) · [`ForwardDeclarations.h`](../ForwardDeclarations.h.md) · [`EnvironmentMap.h`](EnvironmentMap.h.md) · [`GfxData.h`](GfxData.h.md) · [`SimBuffers.h`](SimBuffers.h.md) · [`Skidmark.h`](Skidmark.h.md)
**Used by** — [`Application.cpp`](../Application.cpp.md) · [`GameContext.cpp`](../GameContext.cpp.md) · [`gameplay/Character.cpp`](../gameplay/Character.cpp.md) · [`gameplay/CharacterFactory.cpp`](../gameplay/CharacterFactory.cpp.md) · [`gameplay/SceneMouse.cpp`](../gameplay/SceneMouse.cpp.md) · [`DustPool.cpp`](DustPool.cpp.md) · [`EnvironmentMap.cpp`](EnvironmentMap.cpp.md) · [`GfxActor.cpp`](GfxActor.cpp.md) · [`GfxScene.cpp`](GfxScene.cpp.md) · [`GfxWater.cpp`](GfxWater.cpp.md) · [`HydraxWater.cpp`](HydraxWater.cpp.md) · [`ShadowManager.cpp`](ShadowManager.cpp.md) · [`Skidmark.cpp`](Skidmark.cpp.md) · [`SkyManager.cpp`](SkyManager.cpp.md) · [`SkyXManager.cpp`](SkyXManager.cpp.md) · [`SurveyMapTextureCreator.cpp`](SurveyMapTextureCreator.cpp.md) · [`gfx/camera/CameraManager.cpp`](camera/CameraManager.cpp.md) · [`gui/GUIManager.cpp`](../gui/GUIManager.cpp.md) · [`gui/OverlayWrapper.cpp`](../gui/OverlayWrapper.cpp.md) · [`gui/panels/GUI_CollisionsDebug.cpp`](../gui/panels/GUI_CollisionsDebug.cpp.md) · [`gui/panels/GUI_DirectionArrow.cpp`](../gui/panels/GUI_DirectionArrow.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](../gui/panels/GUI_SurveyMap.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`main.cpp`](../main.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/ActorSpawnerFlow.cpp`](../physics/ActorSpawnerFlow.cpp.md) · [`physics/air/AirBrake.cpp`](../physics/air/AirBrake.cpp.md) · [`physics/air/TurboJet.cpp`](../physics/air/TurboJet.cpp.md) · [`physics/air/TurboProp.cpp`](../physics/air/TurboProp.cpp.md) · [`physics/collision/Collisions.cpp`](../physics/collision/Collisions.cpp.md) · [`physics/flex/FlexBody.cpp`](../physics/flex/FlexBody.cpp.md) · [`physics/flex/FlexFactory.cpp`](../physics/flex/FlexFactory.cpp.md) · [`physics/flex/FlexMeshWheel.cpp`](../physics/flex/FlexMeshWheel.cpp.md) · [`physics/water/Buoyance.cpp`](../physics/water/Buoyance.cpp.md) · [`physics/water/ScrewProp.cpp`](../physics/water/ScrewProp.cpp.md) · [`physics/water/Wavefield.cpp`](../physics/water/Wavefield.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`terrain/ProceduralRoad.cpp`](../terrain/ProceduralRoad.cpp.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md) · [`terrain/TerrainEditor.cpp`](../terrain/TerrainEditor.cpp.md) · [`terrain/TerrainGeometryManager.cpp`](../terrain/TerrainGeometryManager.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md) · [`utils/MeshObject.cpp`](../utils/MeshObject.cpp.md)
**Tier floor** — T2


## Purpose

The rendering half of the game context. Each frame: `BufferSimulationData` (while physics is synchronised) then `UpdateScene(dt)`. Implementation: [`GfxScene.cpp`](GfxScene.cpp.md).

## State

```text
RECORD GfxScene
  scene_manager; dust pools : map<name, DustPool>
  all actor visuals, live actor visuals (this frame), character visuals
  environment map, game-context sim buffer, skidmark config
  free-beam visuals + next id + grouping node
```

## API

`Init`, `CreateDustPools`, `GetDustPool(name)`, `AdjustParticleSystemTimeFactor(psys)`, `SetParticlesVisible`, free-beam add/modify/remove/next id/update and free-force notifications, `DrawNetLabel`, `UpdateScene`, `ClearScene`, register/remove actor and character visuals, `ForceUpdateSingleGfxActor`, `BufferSimulationData`, accessors, `SpecialGetRotationTo(src, dst)`.
