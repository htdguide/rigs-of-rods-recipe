# source/main/scripting

> Embedded scripting: terrain, vehicle, gadget and user scripts running inside the game with access to most of its state.

[`ScriptEngine`](ScriptEngine.h.md) owns the interpreter and every loaded **script unit** — one per loaded script, each with its own module and globals. Units receive `frameStep(dt)` every frame and, when subscribed via `game.registerForEvent`, `eventCallbackEx(...)` for game events. Most events arrive asynchronously through the message queue (`TRIGGER_EVENT_ASYNC`), so scripts never run in the middle of a physics step. The API scripts see is the `game` object ([`GameScript`](GameScript.h.md)) plus the [bindings](bindings/README.md).

Scripts are loaded through the resource system ([`OgreScriptBuilder`](OgreScriptBuilder.h.md)), so they can live inside zipped mods; they persist data with [`LocalStorage`](LocalStorage.h.md).

## Reading order

1. [`ScriptUtils.h`](ScriptUtils.h.md)
2. [`OgreScriptBuilder`](OgreScriptBuilder.h.md) ([impl](OgreScriptBuilder.cpp.md))
3. [`LocalStorage`](LocalStorage.h.md) ([impl](LocalStorage.cpp.md))
4. [`GameScript`](GameScript.h.md) ([impl](GameScript.cpp.md))
5. [`ScriptEngine`](ScriptEngine.h.md) ([impl](ScriptEngine.cpp.md))
6. [bindings/](bindings/README.md)

## Cycle

`ScriptEngine` owns the `GameScript` object and registers it; `GameScript` calls back into `ScriptEngine` for unit management. Read `GameScript` as the API surface first, then the engine.
