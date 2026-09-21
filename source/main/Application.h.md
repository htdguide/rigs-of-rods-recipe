# source/main/Application.h

> The shared vocabulary of the whole program: message types, app/sim/multiplayer states, format-stable enums, the list of global settings, and the registry of singleton subsystems.

**Needs** — [`utils/BitFlags.h`](utils/BitFlags.h.md) · [`system/CVar.h`](system/CVar.h.md) · [`ForwardDeclarations.h`](ForwardDeclarations.h.md) · [`utils/Str.h`](utils/Str.h.md) · [Seam: Text formatting](../../SYSTEM-REQUIREMENTS.md#seam-text-formatting)
**Used by** — [`AppContext.h`](AppContext.h.md) · [`Application.cpp`](Application.cpp.md) · [`audio/MumbleIntegration.cpp`](audio/MumbleIntegration.cpp.md) · [`audio/MumbleIntegration.h`](audio/MumbleIntegration.h.md) · [`audio/Sound.h`](audio/Sound.h.md) · [`audio/SoundManager.cpp`](audio/SoundManager.cpp.md) · [`audio/SoundManager.h`](audio/SoundManager.h.md) · [`audio/SoundScriptManager.h`](audio/SoundScriptManager.h.md) · [`gameplay/AutoPilot.cpp`](gameplay/AutoPilot.cpp.md) · [`gameplay/AutoPilot.h`](gameplay/AutoPilot.h.md) · [`gameplay/Character.cpp`](gameplay/Character.cpp.md) · [`gameplay/CharacterFactory.cpp`](gameplay/CharacterFactory.cpp.md) · [`gameplay/CharacterFactory.h`](gameplay/CharacterFactory.h.md) · [`gameplay/ChatSystem.h`](gameplay/ChatSystem.h.md) · [`gameplay/Engine.h`](gameplay/Engine.h.md) · [`gameplay/Landusemap.cpp`](gameplay/Landusemap.cpp.md) · [`gameplay/Landusemap.h`](gameplay/Landusemap.h.md) · [`gameplay/RepairMode.h`](gameplay/RepairMode.h.md) · [`gameplay/Replay.cpp`](gameplay/Replay.cpp.md) · [`gameplay/Replay.h`](gameplay/Replay.h.md) · [`gameplay/SceneMouse.cpp`](gameplay/SceneMouse.cpp.md) · [`gameplay/SceneMouse.h`](gameplay/SceneMouse.h.md) · [`gameplay/TorqueCurve.cpp`](gameplay/TorqueCurve.cpp.md) · [`gameplay/TorqueCurve.h`](gameplay/TorqueCurve.h.md) · [`gameplay/TyrePressure.h`](gameplay/TyrePressure.h.md) · [`gameplay/VehicleAI.h`](gameplay/VehicleAI.h.md) · [`gfx/AdvancedScreen.h`](gfx/AdvancedScreen.h.md) · [`gfx/ColoredTextAreaOverlayElement.h`](gfx/ColoredTextAreaOverlayElement.h.md) · [`gfx/ColoredTextAreaOverlayElementFactory.h`](gfx/ColoredTextAreaOverlayElementFactory.h.md) · [`gfx/DustPool.cpp`](gfx/DustPool.cpp.md) · [`gfx/DustPool.h`](gfx/DustPool.h.md) · [`gfx/EnvironmentMap.cpp`](gfx/EnvironmentMap.cpp.md) · [`gfx/GfxWater.h`](gfx/GfxWater.h.md) · [`gfx/MovableText.h`](gfx/MovableText.h.md) · [`gfx/ShadowManager.h`](gfx/ShadowManager.h.md) · [`gfx/Skidmark.cpp`](gfx/Skidmark.cpp.md) · [`gfx/Skidmark.h`](gfx/Skidmark.h.md) · [`gfx/SkyManager.h`](gfx/SkyManager.h.md) · [`gfx/SkyXManager.h`](gfx/SkyXManager.h.md) · [`gfx/SurveyMapTextureCreator.cpp`](gfx/SurveyMapTextureCreator.cpp.md) · [`gfx/SurveyMapTextureCreator.h`](gfx/SurveyMapTextureCreator.h.md) · [`gfx/camera/CameraManager.h`](gfx/camera/CameraManager.h.md) · [`gfx/particle/ExtinguishableFireAffector.cpp`](gfx/particle/ExtinguishableFireAffector.cpp.md) · [`gfx/particle/FireExtinguisherAffector.cpp`](gfx/particle/FireExtinguisherAffector.cpp.md) · [`gui/DashBoardManager.cpp`](gui/DashBoardManager.cpp.md) · [`gui/DashBoardManager.h`](gui/DashBoardManager.h.md) · [`gui/GUIManager.h`](gui/GUIManager.h.md) · [`gui/GUIUtils.h`](gui/GUIUtils.h.md) · [`gui/OverlayWrapper.h`](gui/OverlayWrapper.h.md) · [`gui/panels/GUI_AngelScriptExamples.h`](gui/panels/GUI_AngelScriptExamples.h.md) · [`gui/panels/GUI_CollisionsDebug.cpp`](gui/panels/GUI_CollisionsDebug.cpp.md) · [`gui/panels/GUI_ConsoleView.cpp`](gui/panels/GUI_ConsoleView.cpp.md) · [`gui/panels/GUI_ConsoleView.h`](gui/panels/GUI_ConsoleView.h.md) · [`gui/panels/GUI_ConsoleWindow.h`](gui/panels/GUI_ConsoleWindow.h.md) · [`gui/panels/GUI_FlexbodyDebug.cpp`](gui/panels/GUI_FlexbodyDebug.cpp.md) · [`gui/panels/GUI_FlexbodyDebug.h`](gui/panels/GUI_FlexbodyDebug.h.md) · [`gui/panels/GUI_FrictionSettings.cpp`](gui/panels/GUI_FrictionSettings.cpp.md) · [`gui/panels/GUI_FrictionSettings.h`](gui/panels/GUI_FrictionSettings.h.md) · [`gui/panels/GUI_GameAbout.cpp`](gui/panels/GUI_GameAbout.cpp.md) · [`gui/panels/GUI_GameChatBox.cpp`](gui/panels/GUI_GameChatBox.cpp.md) · [`gui/panels/GUI_GameChatBox.h`](gui/panels/GUI_GameChatBox.h.md) · [`gui/panels/GUI_GameControls.cpp`](gui/panels/GUI_GameControls.cpp.md) · [`gui/panels/GUI_GameMainMenu.cpp`](gui/panels/GUI_GameMainMenu.cpp.md) · [`gui/panels/GUI_GameSettings.h`](gui/panels/GUI_GameSettings.h.md) · [`gui/panels/GUI_LoadingWindow.h`](gui/panels/GUI_LoadingWindow.h.md) · [`gui/panels/GUI_MainSelector.cpp`](gui/panels/GUI_MainSelector.cpp.md) · [`gui/panels/GUI_MainSelector.h`](gui/panels/GUI_MainSelector.h.md) · [`gui/panels/GUI_MessageBox.cpp`](gui/panels/GUI_MessageBox.cpp.md) · [`gui/panels/GUI_MessageBox.h`](gui/panels/GUI_MessageBox.h.md) · [`gui/panels/GUI_MultiplayerClientList.cpp`](gui/panels/GUI_MultiplayerClientList.cpp.md) · [`gui/panels/GUI_MultiplayerSelector.cpp`](gui/panels/GUI_MultiplayerSelector.cpp.md) · [`gui/panels/GUI_MultiplayerSelector.h`](gui/panels/GUI_MultiplayerSelector.h.md) · [`gui/panels/GUI_NodeBeamUtils.cpp`](gui/panels/GUI_NodeBeamUtils.cpp.md) · [`gui/panels/GUI_NodeBeamUtils.h`](gui/panels/GUI_NodeBeamUtils.h.md) · [`gui/panels/GUI_RepositorySelector.cpp`](gui/panels/GUI_RepositorySelector.cpp.md) · [`gui/panels/GUI_RepositorySelector.h`](gui/panels/GUI_RepositorySelector.h.md) · [`gui/panels/GUI_ScriptMonitor.h`](gui/panels/GUI_ScriptMonitor.h.md) · [`gui/panels/GUI_SurveyMap.h`](gui/panels/GUI_SurveyMap.h.md) · [`gui/panels/GUI_TextureToolWindow.cpp`](gui/panels/GUI_TextureToolWindow.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](gui/panels/GUI_TopMenubar.cpp.md) · [`gui/panels/GUI_VehicleInfoTPanel.cpp`](gui/panels/GUI_VehicleInfoTPanel.cpp.md) · [`main.cpp`](main.cpp.md) · [`network/CurlHelpers.cpp`](network/CurlHelpers.cpp.md) · [`network/Network.cpp`](network/Network.cpp.md) · [`network/Network.h`](network/Network.h.md) · [`network/OutGauge.cpp`](network/OutGauge.cpp.md) · [`network/OutGauge.h`](network/OutGauge.h.md) · [`physics/ActorExport.cpp`](physics/ActorExport.cpp.md) · [`physics/ActorManager.cpp`](physics/ActorManager.cpp.md) · [`physics/ActorManager.h`](physics/ActorManager.h.md) · [`physics/ActorSpawner.cpp`](physics/ActorSpawner.cpp.md) · [`physics/ActorSpawner.h`](physics/ActorSpawner.h.md) · [`physics/ApproxMath.h`](physics/ApproxMath.h.md) · [`physics/CmdKeyInertia.cpp`](physics/CmdKeyInertia.cpp.md) · [`physics/Differentials.cpp`](physics/Differentials.cpp.md) · [`physics/Savegame.cpp`](physics/Savegame.cpp.md) · [`physics/SimData.h`](physics/SimData.h.md) · [`physics/SlideNode.cpp`](physics/SlideNode.cpp.md) · [`physics/air/AeroEngine.h`](physics/air/AeroEngine.h.md) · [`physics/air/AirBrake.cpp`](physics/air/AirBrake.cpp.md) · [`physics/air/AirBrake.h`](physics/air/AirBrake.h.md) · [`physics/air/Airfoil.cpp`](physics/air/Airfoil.cpp.md) · [`physics/air/Airfoil.h`](physics/air/Airfoil.h.md) · [`physics/air/TurboJet.cpp`](physics/air/TurboJet.cpp.md) · [`physics/air/TurboJet.h`](physics/air/TurboJet.h.md) · [`physics/air/TurboProp.h`](physics/air/TurboProp.h.md) · [`physics/collision/Collisions.cpp`](physics/collision/Collisions.cpp.md) · [`physics/collision/Collisions.h`](physics/collision/Collisions.h.md) · [`physics/collision/DynamicCollisions.cpp`](physics/collision/DynamicCollisions.cpp.md) · [`physics/collision/PointColDetector.h`](physics/collision/PointColDetector.h.md) · [`physics/flex/FlexAirfoil.h`](physics/flex/FlexAirfoil.h.md) · [`physics/flex/FlexBody.cpp`](physics/flex/FlexBody.cpp.md) · [`physics/flex/FlexBody.h`](physics/flex/FlexBody.h.md) · [`physics/flex/FlexFactory.cpp`](physics/flex/FlexFactory.cpp.md) · [`physics/flex/FlexMesh.h`](physics/flex/FlexMesh.h.md) · [`physics/flex/FlexMeshWheel.cpp`](physics/flex/FlexMeshWheel.cpp.md) · [`physics/flex/FlexObj.h`](physics/flex/FlexObj.h.md) · [`physics/water/Buoyance.cpp`](physics/water/Buoyance.cpp.md) · [`physics/water/Buoyance.h`](physics/water/Buoyance.h.md) · [`physics/water/ScrewProp.cpp`](physics/water/ScrewProp.cpp.md) · [`physics/water/ScrewProp.h`](physics/water/ScrewProp.h.md) · [`resources/CacheSystem.h`](resources/CacheSystem.h.md) · [`resources/ContentManager.cpp`](resources/ContentManager.cpp.md) · [`resources/ContentManager.h`](resources/ContentManager.h.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.cpp`](resources/addonpart_fileformat/AddonPartFileFormat.cpp.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.h`](resources/addonpart_fileformat/AddonPartFileFormat.h.md) · [`resources/otc_fileformat/OTCFileFormat.cpp`](resources/otc_fileformat/OTCFileFormat.cpp.md) · [`resources/rig_def_fileformat/RigDef_File.h`](resources/rig_def_fileformat/RigDef_File.h.md) · [`resources/rig_def_fileformat/RigDef_Node.cpp`](resources/rig_def_fileformat/RigDef_Node.cpp.md) · [`resources/rig_def_fileformat/RigDef_Parser.cpp`](resources/rig_def_fileformat/RigDef_Parser.cpp.md) · [`resources/rig_def_fileformat/RigDef_SequentialImporter.cpp`](resources/rig_def_fileformat/RigDef_SequentialImporter.cpp.md) · [`resources/rig_def_fileformat/RigDef_Validator.cpp`](resources/rig_def_fileformat/RigDef_Validator.cpp.md) · [`resources/skin_fileformat/SkinFileFormat.cpp`](resources/skin_fileformat/SkinFileFormat.cpp.md) · [`resources/skin_fileformat/SkinFileFormat.h`](resources/skin_fileformat/SkinFileFormat.h.md) · [`resources/tuneup_fileformat/TuneupFileFormat.cpp`](resources/tuneup_fileformat/TuneupFileFormat.cpp.md) · [`resources/tuneup_fileformat/TuneupFileFormat.h`](resources/tuneup_fileformat/TuneupFileFormat.h.md) · [`scripting/GameScript.h`](scripting/GameScript.h.md) · [`scripting/LocalStorage.cpp`](scripting/LocalStorage.cpp.md) · [`scripting/LocalStorage.h`](scripting/LocalStorage.h.md) · [`scripting/OgreScriptBuilder.cpp`](scripting/OgreScriptBuilder.cpp.md) · [`scripting/OgreScriptBuilder.h`](scripting/OgreScriptBuilder.h.md) · [`scripting/ScriptEngine.cpp`](scripting/ScriptEngine.cpp.md) · [`scripting/ScriptEngine.h`](scripting/ScriptEngine.h.md) · [`scripting/bindings/AircraftEngineAngelscript.cpp`](scripting/bindings/AircraftEngineAngelscript.cpp.md) · [`scripting/bindings/AutopilotAngelscript.cpp`](scripting/bindings/AutopilotAngelscript.cpp.md) · [`scripting/bindings/DashBoardManagerAngelscript.cpp`](scripting/bindings/DashBoardManagerAngelscript.cpp.md) · [`scripting/bindings/MsgQueueAngelscript.cpp`](scripting/bindings/MsgQueueAngelscript.cpp.md) · [`scripting/bindings/OgreAngelscript.cpp`](scripting/bindings/OgreAngelscript.cpp.md) · [`scripting/bindings/ScrewpropAngelscript.cpp`](scripting/bindings/ScrewpropAngelscript.cpp.md) · [`scripting/bindings/ScriptEventsAngelscript.cpp`](scripting/bindings/ScriptEventsAngelscript.cpp.md) · [`scripting/bindings/TerrainAngelscript.cpp`](scripting/bindings/TerrainAngelscript.cpp.md) · [`scripting/bindings/TurbojetAngelscript.cpp`](scripting/bindings/TurbojetAngelscript.cpp.md) · [`scripting/bindings/TurbopropAngelscript.cpp`](scripting/bindings/TurbopropAngelscript.cpp.md) · [`system/AppConfig.cpp`](system/AppConfig.cpp.md) · [`system/CVar.cpp`](system/CVar.cpp.md) · [`system/Console.cpp`](system/Console.cpp.md) · [`system/ConsoleCmd.cpp`](system/ConsoleCmd.cpp.md) · [`system/ConsoleCmd.h`](system/ConsoleCmd.h.md) · [`terrain/ProceduralManager.cpp`](terrain/ProceduralManager.cpp.md) · [`terrain/ProceduralManager.h`](terrain/ProceduralManager.h.md) · [`terrain/ProceduralRoad.cpp`](terrain/ProceduralRoad.cpp.md) · [`terrain/ProceduralRoad.h`](terrain/ProceduralRoad.h.md) · [`terrain/Terrain.h`](terrain/Terrain.h.md) · [`terrain/TerrainEditor.h`](terrain/TerrainEditor.h.md) · [`terrain/TerrainGeometryManager.cpp`](terrain/TerrainGeometryManager.cpp.md) · [`terrain/TerrainGeometryManager.h`](terrain/TerrainGeometryManager.h.md) · [`terrain/TerrainObjectManager.cpp`](terrain/TerrainObjectManager.cpp.md) · [`terrain/TerrainObjectManager.h`](terrain/TerrainObjectManager.h.md) · [`threadpool/ThreadPool.h`](threadpool/ThreadPool.h.md) · [`utils/ConfigFile.cpp`](utils/ConfigFile.cpp.md) · [`utils/ErrorUtils.h`](utils/ErrorUtils.h.md) · [`utils/ForceFeedback.cpp`](utils/ForceFeedback.cpp.md) · [`utils/GenericFileFormat.cpp`](utils/GenericFileFormat.cpp.md) · [`utils/GenericFileFormat.h`](utils/GenericFileFormat.h.md) · [`utils/ImprovedConfigFile.h`](utils/ImprovedConfigFile.h.md) · [`utils/InputEngine.cpp`](utils/InputEngine.cpp.md) · [`utils/InputEngine.h`](utils/InputEngine.h.md) · [`utils/InterThreadStoreVector.h`](utils/InterThreadStoreVector.h.md) · [`utils/Language.cpp`](utils/Language.cpp.md) · [`utils/Language.h`](utils/Language.h.md) · [`utils/MeshObject.h`](utils/MeshObject.h.md) · [`utils/PlatformUtils.cpp`](utils/PlatformUtils.cpp.md) · [`utils/SHA1.h`](utils/SHA1.h.md) · [`utils/Utils.h`](utils/Utils.h.md) · [`utils/WriteTextToTexture.cpp`](utils/WriteTextToTexture.cpp.md) · [`utils/WriteTextToTexture.h`](utils/WriteTextToTexture.h.md)
**Tier floor** — T4: declarations only

## Purpose

Every module includes this file; it is the hub that breaks the dependency cycles of the codebase (see the root [README](../../README.md)). It declares, but does not implement, four things: the **message-queue vocabulary** that decouples subsystems, the **program state machines**, a set of **enums whose numeric or character values are part of file formats**, and the **global settings and service registry** (`App::`). Implementations are in [`Application.cpp`](Application.cpp.md); setting defaults are in [`system/CVar.cpp`](system/CVar.cpp.md).

## State

Stateless (declarations only). The storage lives in [`Application.cpp`](Application.cpp.md).

## `MsgType` — the message queue vocabulary

**Contract** — every cross-subsystem request is a message `{type, description text, payload}` pushed onto the queue owned by [`GameContext`](GameContext.h.md) and drained once per frame by [`main.cpp`](main.cpp.md). Messages that carry a payload **transfer ownership** of it to the consumer (marked "owner" below); a few carry a borrowed reference ("weak").

Groups, in declaration order (the numeric order is exported to scripts, so a rebuild must keep it):

| Group | Messages | Payload |
|---|---|---|
| App | `SHUTDOWN`, `SCREENSHOT`, `DISPLAY_FULLSCREEN`, `DISPLAY_WINDOWED`, `MODCACHE_LOAD`, `MODCACHE_UPDATE`, `MODCACHE_PURGE`, `LOAD_SCRIPT`, `UNLOAD_SCRIPT`, `SCRIPT_THREAD_STATUS`, `REINIT_INPUT` (all `_REQUESTED` except status) | load-script request · script unit id · script event args |
| Net | `CONNECT_REQUESTED/STARTED/PROGRESS/SUCCESS/FAILURE`, `SERVER_KICK`, `DISCONNECT_REQUESTED`, `USER_DISCONNECT`, `RECV_ERROR`, `REFRESH_SERVERLIST_SUCCESS/FAILURE`, `REFRESH_REPOLIST_SUCCESS`, `OPEN_RESOURCE_SUCCESS`, `REFRESH_REPOLIST_FAILURE`, `FETCH_AI_PRESETS_SUCCESS`, `DOWNLOAD_REPOIMAGE_SUCCESS/FAILURE`, `FETCH_AI_PRESETS_FAILURE`, `ADD/REMOVE_PEEROPTIONS_REQUESTED`, `DOWNLOAD_REPOFILE_REQUESTED/SUCCESS/FAILURE/PROGRESS` | server list · curl failure info · repo collection · JSON text · image/file request · peer-options request · progress int |
| Sim | `PAUSE`, `UNPAUSE`, `LOAD_TERRN`, `LOAD_SAVEGAME`, `UNLOAD_TERRN`, `SPAWN_ACTOR`, `MODIFY_ACTOR`, `DELETE_ACTOR`, `SEAT_PLAYER`, `TELEPORT_PLAYER`, `HIDE/UNHIDE/MUTE/UNMUTE_NET_ACTOR`, `SCRIPT_EVENT_TRIGGERED`, `SCRIPT_CALLBACK_QUEUED`, `ACTOR_LINKING`, `ADD/MODIFY/REMOVE_FREEFORCE` | spawn/modify request · actor handle · position · script args · linking request · free-force request/id |
| GUI | `OPEN_MENU`, `CLOSE_MENU`, `OPEN_SELECTOR`, `CLOSE_SELECTOR`, `MP_CLIENTS_REFRESH`, `SHOW/HIDE_MESSAGE_BOX`, `REFRESH_TUNING_MENU`, `SHOW_CHATBOX`, `OPEN_MP_SETTINGS` | loader type + optional GUID · message-box config · chat prefill text |
| Editing | `MODIFY_GROUNDMODEL` (weak), `ENTER/LEAVE_TERRN_EDITOR`, `SAVE_TERRN_CHANGES`, `LOAD/RELOAD/UNLOAD_BUNDLE`, `DELETE_BUNDLE`, `CREATE/MODIFY/DELETE_PROJECT`, `ADD/MODIFY/DELETE_FREEBEAMGFX` | cache entry · filename · project request · free-beam gfx request/id |

`MsgTypeToString` returns the enum's own name as text (used in logs and exception reports).

**Notes** — adding a message is a five-place change in the original (name table, script registration, script push allow-list, script docs, handler). A rebuild should make the name table and script registration derive from one declaration.

## `RigDef::Keyword`

**Contract** — the list of section and directive keywords of the truck file format, `INVALID = 0` then `ADD_ANIMATION = 1` onward in alphabetical order. The parser's keyword-recognizer relies on these numeric values (see [`RigDef_Regexes.h`](resources/rig_def_fileformat/RigDef_Regexes.h.md)); keep them in lock-step. Full list and meanings are in the [rig_def chapter](resources/rig_def_fileformat/README.md). `KeywordToString` gives the lowercase file spelling (`nodes2`, `set_beam_defaults_scale`).

## State machines

```text
ENUM AppState  = BOOTSTRAP | MAIN_MENU | SIMULATION | SHUTDOWN | PRINT_HELP_EXIT | PRINT_VERSION_EXIT
ENUM MpState   = DISABLED | CONNECTING | CONNECTED
ENUM SimState  = OFF | RUNNING | PAUSED | EDITOR_MODE
```

Each is stored in a CVar (`app_state`, `mp_state`, `sim_state`) so scripts and the console can read it. Only the message handlers in [`main.cpp`](main.cpp.md) change them.

## Settings enums

User-facing choices, each with a localized display name (`ToLocalizedString`). In memory they are stored as integers in their CVar, so the order below is part of the script API and the console; in `RoR.cfg` most of them are written as fixed English labels instead (see [`system/AppConfig.cpp`](system/AppConfig.cpp.md)):

| Enum | Values in order |
|---|---|
| `SimGearboxMode` | AUTO, SEMI_AUTO, MANUAL (sequential), MANUAL_STICK, MANUAL_RANGES |
| `GfxShadowType` | NONE, PSSM |
| `GfxExtCamMode` | NONE, STATIC, PITCHING |
| `GfxTexFilter` | NONE, BILINEAR, TRILINEAR, ANISOTROPIC |
| `GfxVegetation` | NONE, 20 %, 50 %, FULL |
| `GfxFlaresMode` | NONE, NO_LIGHTSOURCES, CURR_VEHICLE_HEAD_ONLY, ALL_VEHICLES_HEAD_ONLY, ALL_VEHICLES_ALL_LIGHTS |
| `GfxWaterMode` | NONE, BASIC, REFLECT, FULL_FAST, FULL_HQ, HYDRAX |
| `GfxSkyMode` | SANDSTORM (static), CAELUM, SKYX |
| `EfxReverbEngine` | NONE, REVERB, EAXREVERB |
| `IoInputGrabMode` | NONE, ALL, DYNAMIC |
| `SimResetMode` | HARD = 0, SOFT = 1 |
| `UiPreset` | NOVICE, REGULAR, EXPERT, MINIMALLIST (indexes a preset table in [`GUIManager`](gui/GUIManager.cpp.md)) |

## Format-stable enums

These values are written into and read from content files and must not change:

```text
ENUM FlareType (char)       NONE=0, HEADLIGHT='f', HIGH_BEAM='h', FOG_LIGHT='g', TAIL_LIGHT='t',
                            BRAKE_LIGHT='b', REVERSE_LIGHT='R', SIDELIGHT='s', BLINKER_LEFT='l',
                            BLINKER_RIGHT='r', USER='u', DASHBOARD='d'
ENUM WheelBraking (int)     NONE=0, FOOT_HAND=1, FOOT_HAND_SKID_LEFT=2, FOOT_HAND_SKID_RIGHT=3, FOOT_ONLY=4
ENUM WheelPropulsion (int)  NONE=0, FORWARD=1, BACKWARD=2
ENUM WheelSide (char)       INVALID='n', RIGHT='r', LEFT='l'
ENUM ExtCameraMode (int)    INVALID=-1, CLASSIC=0, CINECAM=1, NODE=2
CameraMode_t (int)          >=0 cinecam index; ALWAYS_HIDDEN=-3, ALWAYS_VISIBLE=-2, 3RDPERSON_ONLY=-1
ENUM VideoCamRole (int)     VIDEOCAM=-1, TRACKING_VIDEOCAM=0, MIRROR=1, MIRROR_NOFLIP=2   # file values
                            TRACKING_MIRROR=-1001, TRACKING_MIRROR_NOFLIP=-1002,
                            MIRROR_PROP_LEFT=-2001, MIRROR_PROP_RIGHT=-2002, INVALID=-9999   # internal
```

`WheelBraking` encodes three independent brakes: the foot brake, the hand/parking brake, and a directional brake that engages only while steering to one side (skid-steer).

## `TObjSpecialObject`

Kinds of placement line in a `.tobj` terrain object file: NONE, TRUCK, LOAD, MACHINE, BOAT, TRUCK2 (spawned at its given height rather than dropped to ground/water) — these five are visible to scripts — then GRID and six road kinds (ROAD, ROAD_BORDER_LEFT/RIGHT/BOTH, ROAD_BRIDGE_NO_PILLARS, ROAD_BRIDGE). `TObjSpecialObjectToString` gives the file spelling (`truck`, `truck2`, `roadborderleft`, `roadbridgenopillar`, …).

## `LoaderType` and `CacheCategoryId`

**Contract** — `LoaderType` is the query mode of the mod cache and the mode of the selector UI: None, Terrain, Vehicle, Truck, Car, Boat, Airplane, Trailer, Train, Load, Extension, Skin, AllBeam, AddonPart, Tuneup, AssetPack, DashBoard, Gadget. Each maps to a set of file extensions (e.g. Vehicle → `truck car`; Extension → `trailer load`; AllBeam → `truck car boat airplane train load`). `CacheCategoryId` numbers categories: dashboards 200–203, gadgets 300–302, projects 8000, tuneups 8001; mod files may use any number below 9000; 9990+ are UI pseudo-categories (Unsorted, All, Fresh, Hidden, SearchResults).

## `VisibilityMasks`

Bit flags on scene objects: bit 1 visible to depth-map passes, bit 2 hidden from depth maps, bit 3 hidden from mirror cameras.

## `App::` settings

**Contract** — about 200 named global settings (CVars), each a typed, persisted, console-editable value. They are the program's configuration *and* its cross-module blackboard. Names are grouped by prefix:

| Prefix | Meaning | Examples |
|---|---|---|
| `app_` | program | `app_state`, `app_language`, `app_async_physics`, `app_num_workers`, `app_screenshot_format`, `app_disable_online_api`, `app_custom_scripts` |
| `sim_` | simulation | `sim_state`, `sim_terrain_name`, `sim_replay_enabled/length/stepping`, `sim_gearbox_mode`, `sim_soft_reset_mode`, `sim_no_collisions`, `sim_live_repair_interval`, `sim_tuning_enabled` |
| `mp_` | multiplayer | `mp_state`, `mp_server_host/port/password`, `mp_player_name/token`, `mp_api_url`, `mp_pseudo_collisions`, `mp_hide_net_labels` |
| `remote_` | online API | `remote_query_url`, `remote_cors_proxy` |
| `diag_` | diagnostics & presets | `diag_preset_terrain/vehicle/spawn_pos`, `diag_log_beam_break/deform/trigger`, `diag_hide_nodes`, `diag_profiler_enabled/rate` |
| `sys_` | directories (set at startup, not persisted) | `sys_process_dir`, `sys_user_dir`, `sys_config_dir`, `sys_cache_dir`, `sys_logs_dir`, `sys_resources_dir`, `sys_savegames_dir`, `sys_screenshot_dir`, `sys_scripts_dir`, `sys_projects_dir`, `sys_thumbnails_dir`, `sys_repo_attachments_dir`, `sys_profiler_dir` |
| `cli_` | command-line overrides (not persisted) | `cli_server_host`, `cli_preset_vehicle`, `cli_resume_autosave`, `cli_custom_scripts` |
| `io_` | input/output devices | `io_analog_smoothing/sensitivity`, `io_ffb_*`, `io_input_grab_mode`, `io_arcade_controls`, `io_outgauge_*`, `io_discord_rpc`, `io_invert_orbitcam` |
| `audio_` | sound | `audio_master_volume`, `audio_enable_efx/occlusion/obstruction`, `audio_efx_reverb_engine`, `audio_doppler_factor`, `audio_menu_music` |
| `gfx_` | graphics | `gfx_water_mode`, `gfx_sky_mode`, `gfx_shadow_type`, `gfx_fov_external/internal`, `gfx_fps_limit`, `gfx_envmap_rate`, `gfx_flexbody_cache`, `gfx_skidmarks_mode`, `gfx_sight_range` |
| `flexbody_defrag_` | mesh-defragmentation tuning for flexbodies | `…_enabled`, `…_const_penalty`, `…_reorder_indices` |
| `ui_` | GUI | `ui_preset`, `ui_hide_gui`, `ui_default_truck_dash`, `ui_default_boat_dash` |

Full list with types, defaults and persistence flags: [`system/CVar.cpp`](system/CVar.cpp.md).

## `App::` service registry

**Contract** — getters for the long-lived singletons: AppContext, ContentManager, OverlayWrapper, GUIManager, Console, InputEngine, CacheSystem, MumbleIntegration, ThreadPool, CameraManager, GfxScene, SoundScriptManager, LanguageEngine, ScriptEngine, Network, GameContext, OutGauge, DiscordRpc. Some are created explicitly and in order at startup (`Create…`), some exist from program start; a getter may return "absent" for a feature compiled out (audio, scripting, multiplayer). Only the overlay wrapper and input engine are ever destroyed and recreated at runtime (input re-init, display switch).

**Notes** — a rebuild can replace this with dependency injection; what must survive is the *creation order* (see [`main.cpp`](main.cpp.md)) and the rule that optional subsystems are absent rather than stubbed, so callers check before use.

## `HandleGenericException` / `HandleMsgQueueException`

**Contract** — called from inside a catch-all block; re-raises the in-flight error to classify it (engine error, standard error, unknown) and then, per flags, writes `"<from>: <message>"` to the log, to the in-game console, and/or forwards it to scripts as a "generic exception caught" event. The message-queue variant uses the message type's name as `from` and reports to console + scripts. Default flags: log + script event.

## `Log` / `LogFormat`

**Contract** — append one line to `RoR.log`. `LogFormat` is printf-style and truncates at ~2000 bytes.

## Constants

- Resource group names: `Temp`, `Cache`, `Thumbnails`, `RepoAttachments`, `Config`, `Content`, `Savegames`, `ManagedMaterials`, `Scripts`, `Logs` — the named search scopes of the resource system.
- `CHARACTER_ANIM_NAME_LEN = 10` — character animation names are truncated to 10 bytes because they are sent over the network.
