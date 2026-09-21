# source/main/scripting/ScriptEngine.h

> Hosts every running script as a "script unit": loading, per-frame stepping, event dispatch, and runtime editing of script globals.

**Needs** — [`bindings/AngelScriptBindings.h`](bindings/AngelScriptBindings.h.md) · [`Application.h`](../Application.h.md) · [`GameContext.h`](../GameContext.h.md) · [`GameScript.h`](GameScript.h.md) · [`utils/InterThreadStoreVector.h`](../utils/InterThreadStoreVector.h.md) · [`gameplay/ScriptEvents.h`](../gameplay/ScriptEvents.h.md) · [Seam: Script engine](../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`Application.cpp`](../Application.cpp.md) · [`GameContext.cpp`](../GameContext.cpp.md) · [`gameplay/Engine.cpp`](../gameplay/Engine.cpp.md) · [`gameplay/SceneMouse.cpp`](../gameplay/SceneMouse.cpp.md) · [`gfx/particle/ExtinguishableFireAffector.cpp`](../gfx/particle/ExtinguishableFireAffector.cpp.md) · [`gui/DashBoardManager.cpp`](../gui/DashBoardManager.cpp.md) · [`gui/panels/GUI_AngelScriptExamples.cpp`](../gui/panels/GUI_AngelScriptExamples.cpp.md) · [`gui/panels/GUI_MainSelector.cpp`](../gui/panels/GUI_MainSelector.cpp.md) · [`gui/panels/GUI_MessageBox.cpp`](../gui/panels/GUI_MessageBox.cpp.md) · [`gui/panels/GUI_ScriptMonitor.cpp`](../gui/panels/GUI_ScriptMonitor.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`main.cpp`](../main.cpp.md) · [`network/Network.cpp`](../network/Network.cpp.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/ActorForcesEuler.cpp`](../physics/ActorForcesEuler.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/air/TurboProp.cpp`](../physics/air/TurboProp.cpp.md) · [`physics/collision/Collisions.cpp`](../physics/collision/Collisions.cpp.md) · [`resources/CacheSystem.cpp`](../resources/CacheSystem.cpp.md) · [`GameScript.cpp`](GameScript.cpp.md) · [`ScriptEngine.cpp`](ScriptEngine.cpp.md) · [`ScriptUtils.h`](ScriptUtils.h.md) · [`scripting/bindings/ActorAngelscript.cpp`](bindings/ActorAngelscript.cpp.md) · [`scripting/bindings/AircraftEngineAngelscript.cpp`](bindings/AircraftEngineAngelscript.cpp.md) · [`scripting/bindings/AutopilotAngelscript.cpp`](bindings/AutopilotAngelscript.cpp.md) · [`scripting/bindings/CacheSystemAngelscript.cpp`](bindings/CacheSystemAngelscript.cpp.md) · [`scripting/bindings/DashBoardManagerAngelscript.cpp`](bindings/DashBoardManagerAngelscript.cpp.md) · [`scripting/bindings/GameScriptAngelscript.cpp`](bindings/GameScriptAngelscript.cpp.md) · [`scripting/bindings/ImGuiAngelscript.cpp`](bindings/ImGuiAngelscript.cpp.md) · [`scripting/bindings/InputEngineAngelscript.cpp`](bindings/InputEngineAngelscript.cpp.md) · [`scripting/bindings/OgreAngelscript.cpp`](bindings/OgreAngelscript.cpp.md) · [`scripting/bindings/ProceduralRoadAngelscript.cpp`](bindings/ProceduralRoadAngelscript.cpp.md) · [`scripting/bindings/ScrewpropAngelscript.cpp`](bindings/ScrewpropAngelscript.cpp.md) · [`scripting/bindings/SoundScriptAngelscript.cpp`](bindings/SoundScriptAngelscript.cpp.md) · [`scripting/bindings/TerrainAngelscript.cpp`](bindings/TerrainAngelscript.cpp.md) · [`scripting/bindings/TurbojetAngelscript.cpp`](bindings/TurbojetAngelscript.cpp.md) · [`scripting/bindings/TurbopropAngelscript.cpp`](bindings/TurbopropAngelscript.cpp.md) · [`system/ConsoleCmd.cpp`](../system/ConsoleCmd.cpp.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md)
**Tier floor** — T2


## Purpose

The game's only door into the embedded script language. Terrains, vehicles, gadgets and the user all load scripts through it. Each load becomes an independent unit with its own module and globals; units share one engine, one execution context and the bound game API ([`GameScript`](GameScript.h.md) plus the [bindings](bindings/README.md)). Compiled only with scripting support; without it, `TRIGGER_EVENT_ASYNC` is a no-op. Implementation: [`ScriptEngine.cpp`](ScriptEngine.cpp.md).

## State

```text
ENUM ScriptCategory: INVALID, ACTOR (from a vehicle file; global `thisActor`), TERRAIN (from the .terrn2; gets eventbox callbacks),
                     GADGET (from a .gadget mod), CUSTOM (console `loadscript`, config `app_custom_scripts`, `-runscript`)
RECORD ScriptUnit
  uniqueId (sequential from 0), category, eventMask (which script events it receives; 0 at load)
  module; resolved entry points: frameStep, eventCallback, eventCallbackEx, defaultEventCallback (each optional)
  associated actor (ACTOR), originating gadget cache entry (GADGET)
  scriptName (.as file name), scriptHash (TERRAIN only), scriptBuffer (source when loaded from memory)
RECORD LoadScriptRequest: filename, buffer, category (default TERRAIN), associated actor id
RECORD ScriptEngine
  engine, one shared context, script log file "Angelscript.log"
  game API object; units : map<id, ScriptUnit>
  terrain unit id (or INVALID); currently executing unit id; currently executing event
  events enabled (cleared for fast shutdown); cross-thread string execution queue
```

Unit id constants: INVALID = −1; DEFAULT = −2 ("the terrain script" — every edit function defaults to it).

**Return codes** — engine codes pass through unchanged (0 success, negatives as the script library defines); the game adds UNSPECIFIED_ERROR −1001, ENGINE_NOT_CREATED −1002, CONTEXT_NOT_CREATED −1003, SCRIPTUNIT_NOT_EXISTS −1004, SCRIPTUNIT_NO_MODULE −1005, FUNCTION_NOT_EXISTS −1006.

**Entry-point signatures** a script may define (all optional): `void main()`, `void frameStep(float)`, `void eventCallback(int, int)`, `void eventCallbackEx(scriptEvents, int, int, int, int, string, string, string, string)`, `void defaultEventCallback(int, string, string, int)` (eventbox trigger type, object instance, box name, node).

## `TRIGGER_EVENT_ASYNC(event, int args…, string args…)`

Posts the event to the game's message queue instead of dispatching immediately; safe from any thread and from inside script execution. The message handler later calls `triggerEvent`.

## API

`loadScript(file, category, actor?, buffer?) → id | INVALID` · `unloadScript(id)` · `framestep(dt)` · `triggerEvent(event, args…)` · `executeString(code)` · `queueStringForExecution(code)` (any thread) · `addFunction` / `functionExists` / `deleteFunction` / `addVariable` / `variableExists` / `deleteVariable` / `getVariable(name, out, type)` on a unit · `getFunctionByDeclAndLogCandidates(unit, flags, name, signature)` · `fireEvent(instance, intensity)` · `envokeCallback(function id, eventsource, node, type)` · `forwardExceptionAsScriptEvent(where)` · `setForwardScriptLogToConsole` · `scriptUnitExists`, `getScriptUnit`, `getTerrainScriptUnit`, `getCurrentlyExecutingScriptUnit`, `getScriptUnits`.
