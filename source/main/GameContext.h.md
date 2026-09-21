# source/main/GameContext.h

> The game state hub: the message queue every subsystem posts to, plus the terrain, actors, characters and player-control state.

**Needs** — [`physics/ActorManager.h`](physics/ActorManager.h.md) · [`resources/CacheSystem.h`](resources/CacheSystem.h.md) · [`gameplay/CharacterFactory.h`](gameplay/CharacterFactory.h.md) · [`gameplay/RaceSystem.h`](gameplay/RaceSystem.h.md) · [`gameplay/RepairMode.h`](gameplay/RepairMode.h.md) · [`gameplay/SceneMouse.h`](gameplay/SceneMouse.h.md) · [`physics/SimData.h`](physics/SimData.h.md) · [`terrain/Terrain.h`](terrain/Terrain.h.md)
**Used by** — [`AppContext.cpp`](AppContext.cpp.md) · [`Application.cpp`](Application.cpp.md) · [`GameContext.cpp`](GameContext.cpp.md) · [`audio/MumbleIntegration.cpp`](audio/MumbleIntegration.cpp.md) · [`audio/SoundManager.h`](audio/SoundManager.h.md) · [`gameplay/AutoPilot.cpp`](gameplay/AutoPilot.cpp.md) · [`gameplay/Character.cpp`](gameplay/Character.cpp.md) · [`gameplay/Landusemap.cpp`](gameplay/Landusemap.cpp.md) · [`gameplay/RaceSystem.cpp`](gameplay/RaceSystem.cpp.md) · [`gameplay/RepairMode.cpp`](gameplay/RepairMode.cpp.md) · [`gameplay/Replay.cpp`](gameplay/Replay.cpp.md) · [`gameplay/SceneMouse.cpp`](gameplay/SceneMouse.cpp.md) · [`gameplay/VehicleAI.cpp`](gameplay/VehicleAI.cpp.md) · [`gfx/DustPool.cpp`](gfx/DustPool.cpp.md) · [`gfx/EnvironmentMap.cpp`](gfx/EnvironmentMap.cpp.md) · [`gfx/GfxActor.cpp`](gfx/GfxActor.cpp.md) · [`gfx/GfxScene.cpp`](gfx/GfxScene.cpp.md) · [`gfx/HydraxWater.cpp`](gfx/HydraxWater.cpp.md) · [`gfx/SkyManager.cpp`](gfx/SkyManager.cpp.md) · [`gfx/SkyXManager.cpp`](gfx/SkyXManager.cpp.md) · [`gfx/SurveyMapTextureCreator.cpp`](gfx/SurveyMapTextureCreator.cpp.md) · [`gfx/camera/CameraManager.cpp`](gfx/camera/CameraManager.cpp.md) · [`gui/GUIManager.cpp`](gui/GUIManager.cpp.md) · [`gui/OverlayWrapper.cpp`](gui/OverlayWrapper.cpp.md) · [`gui/panels/GUI_CollisionsDebug.cpp`](gui/panels/GUI_CollisionsDebug.cpp.md) · [`gui/panels/GUI_FlexbodyDebug.cpp`](gui/panels/GUI_FlexbodyDebug.cpp.md) · [`gui/panels/GUI_FrictionSettings.cpp`](gui/panels/GUI_FrictionSettings.cpp.md) · [`gui/panels/GUI_GameMainMenu.cpp`](gui/panels/GUI_GameMainMenu.cpp.md) · [`gui/panels/GUI_GameSettings.cpp`](gui/panels/GUI_GameSettings.cpp.md) · [`gui/panels/GUI_MainSelector.cpp`](gui/panels/GUI_MainSelector.cpp.md) · [`gui/panels/GUI_MessageBox.cpp`](gui/panels/GUI_MessageBox.cpp.md) · [`gui/panels/GUI_MultiplayerClientList.cpp`](gui/panels/GUI_MultiplayerClientList.cpp.md) · [`gui/panels/GUI_MultiplayerSelector.cpp`](gui/panels/GUI_MultiplayerSelector.cpp.md) · [`gui/panels/GUI_NodeBeamUtils.cpp`](gui/panels/GUI_NodeBeamUtils.cpp.md) · [`gui/panels/GUI_RepositorySelector.cpp`](gui/panels/GUI_RepositorySelector.cpp.md) · [`gui/panels/GUI_ScriptMonitor.cpp`](gui/panels/GUI_ScriptMonitor.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](gui/panels/GUI_SurveyMap.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](gui/panels/GUI_TopMenubar.cpp.md) · [`gui/panels/GUI_VehicleInfoTPanel.cpp`](gui/panels/GUI_VehicleInfoTPanel.cpp.md) · [`main.cpp`](main.cpp.md) · [`network/CurlHelpers.cpp`](network/CurlHelpers.cpp.md) · [`network/CurlHelpers.h`](network/CurlHelpers.h.md) · [`network/Network.cpp`](network/Network.cpp.md) · [`physics/Actor.cpp`](physics/Actor.cpp.md) · [`physics/ActorExport.cpp`](physics/ActorExport.cpp.md) · [`physics/ActorForcesEuler.cpp`](physics/ActorForcesEuler.cpp.md) · [`physics/ActorManager.cpp`](physics/ActorManager.cpp.md) · [`physics/ActorSlideNode.cpp`](physics/ActorSlideNode.cpp.md) · [`physics/ActorSpawner.cpp`](physics/ActorSpawner.cpp.md) · [`physics/Savegame.cpp`](physics/Savegame.cpp.md) · [`physics/collision/Collisions.cpp`](physics/collision/Collisions.cpp.md) · [`physics/collision/DynamicCollisions.cpp`](physics/collision/DynamicCollisions.cpp.md) · [`physics/collision/PointColDetector.cpp`](physics/collision/PointColDetector.cpp.md) · [`physics/water/Buoyance.cpp`](physics/water/Buoyance.cpp.md) · [`physics/water/ScrewProp.cpp`](physics/water/ScrewProp.cpp.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.cpp`](resources/addonpart_fileformat/AddonPartFileFormat.cpp.md) · [`scripting/GameScript.cpp`](scripting/GameScript.cpp.md) · [`scripting/ScriptEngine.cpp`](scripting/ScriptEngine.cpp.md) · [`scripting/ScriptEngine.h`](scripting/ScriptEngine.h.md) · [`system/AppCommandLine.cpp`](system/AppCommandLine.cpp.md) · [`system/ConsoleCmd.cpp`](system/ConsoleCmd.cpp.md) · [`terrain/ProceduralRoad.cpp`](terrain/ProceduralRoad.cpp.md) · [`terrain/TerrainEditor.cpp`](terrain/TerrainEditor.cpp.md) · [`terrain/TerrainObjectManager.cpp`](terrain/TerrainObjectManager.cpp.md) · [`utils/ForceFeedback.cpp`](utils/ForceFeedback.cpp.md)
**Tier floor** — T2


## Purpose

RoR's gameplay is three kinds of object — a static terrain, soft-body actors (vehicles, loads, machines) and characters — and this class owns the relationships between them: which actor the player drives, what the loader is spawning, who receives imported commands. Above all it owns the **message queue**: every structural change (load terrain, spawn/delete actor, seat player, connect, open a window…) is requested as a message and applied by the main loop at a safe point in the frame ([`main.cpp`](main.cpp.md#message-processing)). Implementation: [`GameContext.cpp`](GameContext.cpp.md); savegame methods live in [`physics/Savegame.cpp`](physics/Savegame.cpp.md).

## State

```text
RECORD Message
  type : MsgType; description : text; payload : owned object or none (its type depends on the message)
  chain : list<Message>          # posted right after this message is processed
QUEUE Messages (FIFO, locked — any thread may post)
RECORD GameContext
  queue; pointer to the end of the most recent chain (for ChainMessage)
  terrain (or none); actor manager; player actor; previous player actor; last actor spawned by the user;
  actor receiving imported commands (nearest actor that accepts commands while the player is on foot)
  last user selection (vehicle, skin, tune-up, section config) for "respawn last";
  current loader context (a spawn request being built by the selector); dummy "default skin" entry
  character factory; race system; repair mode; scene mouse; recording timer + previous position
```

**Ordering rule** — `PushMessage` gives no ordering guarantee between messages whose handlers re-queue themselves; when B must run after A, `ChainMessage(B)` attaches it to A's chain. Chains nest: chaining again appends to the chained message.

## API

- Queue: `PushMessage`, `ChainMessage`, `HasMessages`, `PopMessage`.
- Terrain: `LoadTerrain(name)`, `UnloadTerrain`, `GetTerrain`.
- Actors: `SpawnActor(request)`, `ModifyActor(request)`, `DeleteActor`, `UpdateActors`, `GetActorManager`, `FetchPrevVehicleOnList`, `FetchNextVehicleOnList`, `FindActorByCollisionBox`, `RespawnLastActor`, `SpawnPreselectedActor(name, config)`, `GetPlayerActor`, `GetPrevPlayerActor`, `GetLastSpawnedActor`, `GetActorRemotelyReceivingCommands`, `SetPrevPlayerActor`, `ChangePlayerActor`, `ShowLoaderGUI(type, instance, box)`, `OnLoaderGuiCancel`, `OnLoaderGuiApply(type, entry, config)`.
- Characters: `CreatePlayerCharacter`, `GetPlayerCharacter`, `GetCharacterFactory`.
- Savegames: `LoadScene`, `SaveScene`, `GetQuicksaveFilename`, `ExtractSceneName`, `ExtractSceneTerrain`, `HandleSavegameHotkeys`.
- Misc: `GetRaceSystem`, `GetRepairMode`, `GetSceneMouse`, `TeleportPlayer(x, z)`, input handlers `UpdateGlobalInputEvents`, `UpdateSimInputEvents`, `UpdateSkyInputEvents`, `UpdateCommonInputEvents`, `UpdateAirplaneInputEvents`, `UpdateBoatInputEvents`, `UpdateTruckInputEvents`.
