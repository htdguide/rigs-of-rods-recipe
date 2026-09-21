# source/main/scripting/ScriptEngine.cpp

> Unit lifecycle, execution with diagnostics, and event fan-out to units that asked for them.

**Needs** — [`ScriptEngine.h`](ScriptEngine.h.md) · [`Application.h`](../Application.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`physics/collision/Collisions.h`](../physics/collision/Collisions.h.md) · [`system/Console.h`](../system/Console.h.md) · [`GameContext.h`](../GameContext.h.md) · [`GameScript.h`](GameScript.h.md) · [`LocalStorage.h`](LocalStorage.h.md) · [`OgreScriptBuilder.h`](OgreScriptBuilder.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md) · [`gameplay/ScriptEvents.h`](../gameplay/ScriptEvents.h.md) · [`ScriptUtils.h`](ScriptUtils.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [`gameplay/VehicleAI.h`](../gameplay/VehicleAI.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md) · [Seam: Script engine](../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — callers of [`ScriptEngine.h`](ScriptEngine.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`ScriptEngine.h`](ScriptEngine.h.md).

## State

See header.

## Startup

Create the engine allowing unsafe references (the GUI binding passes references to temporaries). Install the compile-message callback. Register add-ons in this order — arrays, strings and string utilities, math, `FLT_MAX` / `INT_MAX`, `any`, dictionary, `log(string)` / `print(string)` (to the script log) — then the game bindings: math types, cache system, local storage, input, GUI, vehicle AI, console, engine, dashboard, turbojet, turboprop, aircraft engine, autopilot, screwprop, actor, procedural roads, terrain, message queue, sound scripts, game API, script events, generic document. Finally the globals `game`, `console`, `inputs`, `modcache`, and one execution context. The order matters because later bindings refer to types registered by earlier ones.

## `loadScript(name, category, actor, buffer)`

```text
ext "gadget": find the gadget in the mod cache (missing → console error, INVALID), load its resources,
              script = "<basename>.as", category = GADGET
ext "as": script = name;  otherwise → console error, INVALID
id = next counter; create unit {name, category, buffer, gadget}; TERRAIN → becomes the terrain unit; ACTOR → store actor
result = setupScriptUnit(id)
add to "recent scripts" (CUSTOM from file, or the gadget file)
failure → remove unit (and terrain id), fire ANGELSCRIPT_MANIPULATIONS(SCRIPT_LOAD_FAILED, category, name), INVALID
success → fire ANGELSCRIPT_MANIPULATIONS(SCRIPT_LOADED, id, category, name), id
```

## `setupScriptUnit(id)`

```text
module name = "<script>(category:<C>,unique ID:<id>)"
new module via the include-aware builder (OgreScriptBuilder)
ACTOR: prepend section "BeamClass@ thisActor;"
prepend "const int thisScript = <id>;"
add the source (from buffer, else the file from resources); build — with "currently executing" = id so compile messages are attributed
TERRAIN: remember the source hash
resolve frameStep / eventCallback / eventCallbackEx / defaultEventCallback by exact signature
IF main() exists:
  ACTOR: set `thisActor` to the actor (the script holds a reference)
  run main(); anything but "finished" fails the load
```

## Execution with diagnostics

Every call into script goes through one path: prepare the function (error → log, skip), install the per-statement line callback if the unit registered ANGELSCRIPT_LINECALLBACK (never while dispatching that event itself, to avoid recursion) and the exception callback if it registered ANGELSCRIPT_EXCEPTIONCALLBACK, mark the unit as executing, run, unmark, clear callbacks. Aborts, script exceptions (with function and line) and "not prepared" are logged.

Diagnostics are surfaced to scripts as asynchronous events: compile messages → ANGELSCRIPT_MSGCALLBACK(unit, type, row, col, section, message); line callback → ANGELSCRIPT_LINECALLBACK(unit, line, stack depth, function, object type, object); exception → ANGELSCRIPT_EXCEPTIONCALLBACK(unit, line, function, message). Native failures inside API calls are caught and forwarded as GENERIC_EXCEPTION_CAUGHT(unit, where, exception kind, description) — the script is not stopped.

## `framestep(dt)`

Drain the cross-thread string queue and execute each string; then call `frameStep(dt)` on every unit that defines it.

## `triggerEvent(event, args…)`

Skipped when events are disabled. For each unit whose eventMask contains the event and which defines `eventCallbackEx` (preferred) or `eventCallback`: call it with (event, arg1) — plus the other seven arguments for the Ex form — recording the event as "currently executing".

## `envokeCallback(function id, eventsource, node, type)` — eventbox callbacks

For each unit: function id ≤ 0 → use the unit's `defaultEventCallback`; call it with (trigger type, object instance name, box name, node or −1).

**Notes** — the explicit id (from a legacy per-object handler) belongs to the terrain module, yet the original tries it on every unit, and preparing a foreign function fails with a logged error. A rebuild dispatches an explicit handler only to its owning unit.

## `fireEvent(instance, intensity)`

Calls `void fireEvent(string, float)` on every unit (a hook for fire/effect scripts). The original does not check that the function exists before preparing it.

## `executeString(code)`

Compiles and runs a snippet in the terrain unit's module (so it sees terrain globals). Returns the engine result.

## Editing globals — `addFunction` … `getVariable`

Each validates engine, context, unit and module first (codes −1002…−1005).

- `addFunction(decl)` compiles and adds the function; if it matches one of the entry-point signatures and the **terrain** unit lacks it, the terrain unit adopts it as that entry point.
- `functionExists(decl)` — see note.
- `deleteFunction(decl)` removes it, runs garbage collection, and clears any terrain entry point that pointed at it.
- `addVariable(decl)`, `variableExists(name)` (0 or negative), `deleteVariable(name)`.
- `getVariable(name, out, requested type)` copies a global out: handle requested → reference cast (const-correct), fails if the cast yields none; object → assign when types are identical; primitive → byte copy when types are identical; otherwise error.

**Notes** — `functionExists` in the original returns FUNCTION_NOT_EXISTS when the function *does* exist and 0 when it doesn't; scripts in the wild may depend on the inversion. Entry-point adoption in `add/deleteFunction` always targets the terrain unit regardless of the unit edited.

## `getFunctionByDeclAndLogCandidates(unit, flags, name, signature)`

Exact lookup of `signature` formatted with `name`. On a miss: if a function of that name exists with another signature, warn (unless SILENT) that the arguments are probably mistyped; if none exists and the flag is REQUIRED, warn that it is missing.

## `unloadScript(id)`

Fire ANGELSCRIPT_MANIPULATIONS(SCRIPT_UNLOADING, id, category, name) synchronously, discard the module, remove the unit; clear the terrain id if it was the terrain unit.

## `setForwardScriptLogToConsole(on)`

Mirror (or stop mirroring) the script log into the in-game console.
