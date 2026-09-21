# source/main/gfx/GfxActor.h

> All visuals of one actor: props, flexbodies, wheels, rods, cab, flares, particles, video cameras, debug views — driven from the actor’s sim buffer.

**Needs** — [`gameplay/AutoPilot.h`](../gameplay/AutoPilot.h.md) · [`physics/Differentials.h`](../physics/Differentials.h.md) · [`ForwardDeclarations.h`](../ForwardDeclarations.h.md) · [`GfxData.h`](GfxData.h.md) · [`resources/rig_def_fileformat/RigDef_Prerequisites.h`](../resources/rig_def_fileformat/RigDef_Prerequisites.h.md) · [`SimBuffers.h`](SimBuffers.h.md) · [`terrain/SurveyMapEntity.h`](../terrain/SurveyMapEntity.h.md) · [`threadpool/ThreadPool.h`](../threadpool/ThreadPool.h.md)
**Used by** — [`EnvironmentMap.cpp`](EnvironmentMap.cpp.md) · [`GfxActor.cpp`](GfxActor.cpp.md) · [`gui/GUIManager.cpp`](../gui/GUIManager.cpp.md) · [`gui/OverlayWrapper.cpp`](../gui/OverlayWrapper.cpp.md) · [`gui/panels/GUI_DirectionArrow.cpp`](../gui/panels/GUI_DirectionArrow.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](../gui/panels/GUI_SurveyMap.cpp.md) · [`gui/panels/GUI_VehicleInfoTPanel.cpp`](../gui/panels/GUI_VehicleInfoTPanel.cpp.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/air/AirBrake.cpp`](../physics/air/AirBrake.cpp.md) · [`physics/air/TurboJet.cpp`](../physics/air/TurboJet.cpp.md) · [`physics/air/TurboProp.cpp`](../physics/air/TurboProp.cpp.md) · [`physics/flex/FlexAirfoil.cpp`](../physics/flex/FlexAirfoil.cpp.md) · [`physics/flex/FlexBody.cpp`](../physics/flex/FlexBody.cpp.md) · [`physics/flex/FlexMesh.cpp`](../physics/flex/FlexMesh.cpp.md) · [`physics/flex/FlexMeshWheel.cpp`](../physics/flex/FlexMeshWheel.cpp.md) · [`physics/flex/FlexObj.cpp`](../physics/flex/FlexObj.cpp.md) · [`resources/CacheSystem.cpp`](../resources/CacheSystem.cpp.md)
**Tier floor** — T2


## Purpose

Built by the [spawner](../physics/ActorSpawner.cpp.md) alongside the actor; updated once per rendered frame by [`GfxScene`](GfxScene.cpp.md) using only the actor's [`ActorSB`](SimBuffers.h.md) snapshot (with a few read-only exceptions). Implementation: [`GfxActor.cpp`](GfxActor.cpp.md).

## State

```text
RECORD GfxActor
  actor, resource group, driver-seat prop index (−1 none), rods parent node
  initialized, video-camera state = ENABLED_ONLINE, debug view = NONE, last debug view = SKELETON,
  beacons active (starts true so the first update switches them off), prop-anim memory (previous crank factor, shift timer, previous gear)
  pending worker tasks: wheel meshes, flexbodies
  nodes[] (NodeGfx), rods[] (BeamGfx), airbrakes[], props[], flexbodies[] (sorted by vertex count, largest first),
  wheels[], video cameras[], flare materials[], custom particles[], exhausts[]
  particle pools: drip, dust (also vapour and tyre smoke), splash, ripple, sparks, clump
  cab: flex object, scene node, entity, visual material, transparent material, plain and emissive templates
  help material/texture, survey-map entity, sim buffer
```

## API groups

Registration (cab material/mesh, flexbody sort), element access, visual state (material flares, cab lights, video-camera state, scale, debug views, beacons, hot nodes, remove rod), visibility toggles (rods, flexbodies, wheels, all, wings, shadows, props, aero engines), per-frame updates (video cameras, particles, rods, wheels, flexbodies, debug view, cab, wings, props & beacons, prop animations, airbrakes, custom particles, exhausts, aero engines, net labels, flares), sim buffer (`UpdateSimDataBuffer`, accessors), worker-task completion (`FinishWheelUpdates`, `FinishFlexbodyTasks`), helpers (`IsActorLive`, `CalculateDriverPos`, `CalcPropAnimation`, beacon count, wheel side/rim name).
