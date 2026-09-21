# source/main/scripting/bindings/AngelScriptBindings.h

> Declares one registration function per script-exposed subsystem.

**Needs** — · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`audio/SoundScriptManager.h`](../../audio/SoundScriptManager.h.md) · [`scripting/ScriptEngine.h`](../ScriptEngine.h.md) · [`ActorAngelscript.cpp`](ActorAngelscript.cpp.md) · [`CacheSystemAngelscript.cpp`](CacheSystemAngelscript.cpp.md) · [`ConsoleAngelscript.cpp`](ConsoleAngelscript.cpp.md) · [`EngineAngelscript.cpp`](EngineAngelscript.cpp.md) · [`GameScriptAngelscript.cpp`](GameScriptAngelscript.cpp.md) · [`GenericFileFormatAngelscript.cpp`](GenericFileFormatAngelscript.cpp.md) · [`LocalStorageAngelscript.cpp`](LocalStorageAngelscript.cpp.md) · [`MsgQueueAngelscript.cpp`](MsgQueueAngelscript.cpp.md) · [`ScriptEventsAngelscript.cpp`](ScriptEventsAngelscript.cpp.md) · [`VehicleAiAngelscript.cpp`](VehicleAiAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

The index of bindings: `RegisterActor`, `RegisterVehicleAi`, `RegisterInputEngine`, `RegisterConsole`, `RegisterLocalStorage`, `RegisterGameScript`, `RegisterScriptEvents`, `RegisterImGuiBindings`, `RegisterOgreObjects`, `RegisterTerrain`, `RegisterProceduralRoad`, `RegisterGenericFileFormat`, `RegisterMessageQueue`, `RegisterSoundScript`, `RegisterCacheSystem`, `RegisterEngine`, `RegisterDashBoardManager`, `RegisterAircraftEngine`, `RegisterTurboprop`, `RegisterTurbojet`, `RegisterAutopilot`, `RegisterScrewprop`. Each takes the script engine and adds types, methods, enums and globals. Call order is set by [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless.
