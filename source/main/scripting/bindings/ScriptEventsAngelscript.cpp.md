# source/main/scripting/bindings/ScriptEventsAngelscript.cpp

> Script enums for events and their sub-codes.

**Needs** — [`gameplay/ScriptEvents.h`](../../gameplay/ScriptEvents.h.md) · [`AngelScriptBindings.h`](AngelScriptBindings.h.md) · [`Application.h`](../../Application.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## Enums

- `scriptEvents` — the event bits: SE_EVENTBOX_ENTER, SE_EVENTBOX_EXIT, SE_TRUCK_ENTER, SE_TRUCK_EXIT, SE_TRUCK_ENGINE_DIED, SE_TRUCK_ENGINE_FIRE, SE_TRUCK_TOUCHED_WATER, SE_TRUCK_LIGHT_TOGGLE, SE_TRUCK_TIE_TOGGLE, SE_TRUCK_PARKINGBRAKE_TOGGLE, SE_TRUCK_BEACONS_TOGGLE, SE_TRUCK_CPARTICLES_TOGGLE, SE_GENERIC_NEW_TRUCK, SE_GENERIC_DELETED_TRUCK, SE_TRUCK_RESET, SE_TRUCK_TELEPORT, SE_TRUCK_MOUSE_GRAB, SE_ANGELSCRIPT_MANIPULATIONS, SE_ANGELSCRIPT_MSGCALLBACK, SE_ANGELSCRIPT_LINECALLBACK, SE_ANGELSCRIPT_EXCEPTIONCALLBACK, SE_ANGELSCRIPT_THREAD_STATUS, SE_GENERIC_MESSAGEBOX_CLICK, SE_GENERIC_EXCEPTION_CAUGHT, SE_GENERIC_MODCACHE_ACTIVITY, SE_GENERIC_TRUCK_LINKING_CHANGED, SE_GENERIC_FREEFORCES_ACTIVITY, SE_ALL_EVENTS, SE_NO_EVENTS. Values come from [`gameplay/ScriptEvents.h`](../../gameplay/ScriptEvents.h.md).
- `angelScriptManipulationType` — CONSOLE_SNIPPET_EXECUTED, SCRIPT_LOADED, SCRIPT_LOAD_FAILED, SCRIPT_UNLOADING, ACTORSIMATTR_SET.
- `angelScriptThreadStatus` — NONE, CURLSTRING_PROGRESS, CURLSTRING_SUCCESS, CURLSTRING_FAILURE.
- `modCacheActivityType` — ENTRY_ADDED, ENTRY_DELETED, BUNDLE_LOADED, BUNDLE_RELOADED, BUNDLE_UNLOADED.
- `freeForcesActivityType` — NONE, ADDED, MODIFIED, REMOVED, DEFORMED, BROKEN.
