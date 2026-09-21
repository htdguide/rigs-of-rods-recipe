# source/main/scripting/bindings

> The script-visible API, one registration file per subsystem.

Everything a mod script can name — types, methods, properties, enums, globals — is declared here and nowhere else. The native classes they wrap are documented in their own chapters; these pages list the *script names*, which existing content depends on. Treat them like a wire format: a rebuild may implement them however it likes but should not rename them.

Registration order matters (a type must exist before a method mentions it) and is fixed in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup): math/scene types first, then content cache, storage, input, GUI, vehicle subsystems, actor, terrain, messages, sound, the `game` object and events.

## Reading order

1. [`AngelScriptBindings.h`](AngelScriptBindings.h.md) — the index
2. [`OgreAngelscript.cpp`](OgreAngelscript.cpp.md) — vectors, quaternions, angles, scene graph
3. [`CacheSystemAngelscript.cpp`](CacheSystemAngelscript.cpp.md), [`LocalStorageAngelscript.cpp`](LocalStorageAngelscript.cpp.md), [`GenericFileFormatAngelscript.cpp`](GenericFileFormatAngelscript.cpp.md)
4. [`InputEngineAngelscript.cpp`](InputEngineAngelscript.cpp.md), [`ImGuiAngelscript.cpp`](ImGuiAngelscript.cpp.md), [`ConsoleAngelscript.cpp`](ConsoleAngelscript.cpp.md)
5. Vehicle subsystems: [`EngineAngelscript.cpp`](EngineAngelscript.cpp.md), [`DashBoardManagerAngelscript.cpp`](DashBoardManagerAngelscript.cpp.md), [`AircraftEngineAngelscript.cpp`](AircraftEngineAngelscript.cpp.md), [`TurbojetAngelscript.cpp`](TurbojetAngelscript.cpp.md), [`TurbopropAngelscript.cpp`](TurbopropAngelscript.cpp.md), [`AutopilotAngelscript.cpp`](AutopilotAngelscript.cpp.md), [`ScrewpropAngelscript.cpp`](ScrewpropAngelscript.cpp.md), [`VehicleAiAngelscript.cpp`](VehicleAiAngelscript.cpp.md)
6. [`ActorAngelscript.cpp`](ActorAngelscript.cpp.md)
7. [`TerrainAngelscript.cpp`](TerrainAngelscript.cpp.md), [`ProceduralRoadAngelscript.cpp`](ProceduralRoadAngelscript.cpp.md)
8. [`MsgQueueAngelscript.cpp`](MsgQueueAngelscript.cpp.md), [`SoundScriptAngelscript.cpp`](SoundScriptAngelscript.cpp.md), [`GameScriptAngelscript.cpp`](GameScriptAngelscript.cpp.md), [`ScriptEventsAngelscript.cpp`](ScriptEventsAngelscript.cpp.md)
