# source/main/system/Console.h

> The in-game console backend: a thread-safe message history, the setting registry, commands, and hooks for the config file and command line.

**Needs** — [`CVar.h`](CVar.h.md) · [`ConsoleCmd.h`](ConsoleCmd.h.md) · [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine) (log listener, timer)
**Used by** — [`AppContext.cpp`](../AppContext.cpp.md) · [`Application.cpp`](../Application.cpp.md) · [`GameContext.cpp`](../GameContext.cpp.md) · [`gameplay/ChatSystem.cpp`](../gameplay/ChatSystem.cpp.md) · [`gameplay/Engine.cpp`](../gameplay/Engine.cpp.md) · [`gameplay/Landusemap.cpp`](../gameplay/Landusemap.cpp.md) · [`gameplay/VehicleAI.cpp`](../gameplay/VehicleAI.cpp.md) · [`gfx/GfxScene.cpp`](../gfx/GfxScene.cpp.md) · [`gfx/camera/CameraManager.cpp`](../gfx/camera/CameraManager.cpp.md) · [`gui/DashBoardManager.cpp`](../gui/DashBoardManager.cpp.md) · [`gui/panels/GUI_ConsoleView.cpp`](../gui/panels/GUI_ConsoleView.cpp.md) · [`gui/panels/GUI_ConsoleView.h`](../gui/panels/GUI_ConsoleView.h.md) · [`gui/panels/GUI_GameChatBox.cpp`](../gui/panels/GUI_GameChatBox.cpp.md) · [`gui/panels/GUI_RepositorySelector.cpp`](../gui/panels/GUI_RepositorySelector.cpp.md) · [`gui/panels/GUI_TextureToolWindow.cpp`](../gui/panels/GUI_TextureToolWindow.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`main.cpp`](../main.cpp.md) · [`network/Network.cpp`](../network/Network.cpp.md) · [`physics/ActorForcesEuler.cpp`](../physics/ActorForcesEuler.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/Savegame.cpp`](../physics/Savegame.cpp.md) · [`physics/flex/FlexBody.cpp`](../physics/flex/FlexBody.cpp.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.cpp`](../resources/addonpart_fileformat/AddonPartFileFormat.cpp.md) · [`resources/otc_fileformat/OTCFileFormat.cpp`](../resources/otc_fileformat/OTCFileFormat.cpp.md) · [`resources/rig_def_fileformat/RigDef_Parser.cpp`](../resources/rig_def_fileformat/RigDef_Parser.cpp.md) · [`resources/rig_def_fileformat/RigDef_Parser.h`](../resources/rig_def_fileformat/RigDef_Parser.h.md) · [`resources/rig_def_fileformat/RigDef_SequentialImporter.cpp`](../resources/rig_def_fileformat/RigDef_SequentialImporter.cpp.md) · [`resources/rig_def_fileformat/RigDef_Validator.cpp`](../resources/rig_def_fileformat/RigDef_Validator.cpp.md) · [`resources/skin_fileformat/SkinFileFormat.cpp`](../resources/skin_fileformat/SkinFileFormat.cpp.md) · [`resources/terrn2_fileformat/Terrn2FileFormat.cpp`](../resources/terrn2_fileformat/Terrn2FileFormat.cpp.md) · [`resources/tuneup_fileformat/TuneupFileFormat.cpp`](../resources/tuneup_fileformat/TuneupFileFormat.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`scripting/OgreScriptBuilder.cpp`](../scripting/OgreScriptBuilder.cpp.md) · [`scripting/ScriptEngine.cpp`](../scripting/ScriptEngine.cpp.md) · [`scripting/bindings/ConsoleAngelscript.cpp`](../scripting/bindings/ConsoleAngelscript.cpp.md) · [`AppCommandLine.cpp`](AppCommandLine.cpp.md) · [`AppConfig.cpp`](AppConfig.cpp.md) · [`CVar.cpp`](CVar.cpp.md) · [`Console.cpp`](Console.cpp.md) · [`ConsoleCmd.cpp`](ConsoleCmd.cpp.md) · [`terrain/ProceduralRoad.cpp`](../terrain/ProceduralRoad.cpp.md) · [`terrain/TerrainEditor.cpp`](../terrain/TerrainEditor.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md) · [`utils/ConfigFile.cpp`](../utils/ConfigFile.cpp.md) · [`utils/ConfigFile.h`](../utils/ConfigFile.h.md) · [`utils/ForceFeedback.cpp`](../utils/ForceFeedback.cpp.md) · [`utils/GenericFileFormat.cpp`](../utils/GenericFileFormat.cpp.md) · [`utils/InputEngine.cpp`](../utils/InputEngine.cpp.md)
**Tier floor** — T2

## Purpose

One object with four jobs, split across four implementation files: message history ([`Console.cpp`](Console.cpp.md)), commands ([`ConsoleCmd.cpp`](ConsoleCmd.cpp.md)), settings ([`CVar.cpp`](CVar.cpp.md)), command line ([`AppCommandLine.cpp`](AppCommandLine.cpp.md)) and config file ([`AppConfig.cpp`](AppConfig.cpp.md)). The GUI frontend is [`GUI_ConsoleView`](../gui/panels/GUI_ConsoleView.h.md); the chat box and on-screen notifications read the same history.

## State

```text
ENUM MessageType = HELP | TITLE | SYSTEM_NOTICE | SYSTEM_ERROR | SYSTEM_WARNING | SYSTEM_REPLY | SYSTEM_NETCHAT
ENUM MessageArea = INFO | LOG | SCRIPT | ACTOR | TERRN

RECORD Message
  area, type
  timestamp_ms : int       # console's own millisecond timer
  net_user_id  : int       # 0 = local/server, >0 = multiplayer user
  text, icon   : text

RECORD Console
  messages      : list<Message>     # guarded by messages_lock; grows without bound until cleared
  messages_lock : mutex
  timer         : monotonic ms timer started at construction
  cvars         : map<name, CVar>
  cvars_by_long : map<long name, CVar>
  commands      : map<name, Command>
```

## Messages

**Contract** — `putMessage(area, type, text, icon)` and `putNetMessage(user_id, type, text)` append to history (and usually the log); `forwardLogMessage(area, text, level)` maps log levels to types; `purgeNetChatMessagesByUser(id)`; `queryMessageTimer()`. Readers take a scoped lock guard that exposes the list while held. Safe from any thread.

## Commands, settings, command line, config

`regBuiltinCommands`, `doCommand(line)`; `cVarCreate/Assign/Find/Set/Get`, `cVarSetupBuiltins`, `getCVars`, `getCommands`; `processCommandLine(argc, argv)`, `showCommandLineUsage`, `showCommandLineVersion`; `loadConfig`, `saveConfig`. See the respective `.cpp` twins.

## Log listener

The console subscribes to the main log; see [`Console.cpp`](Console.cpp.md).
