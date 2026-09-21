# source/main/gameplay/ScriptEvents.h

> The script event catalogue: event bits, their argument conventions, and sub-type enums.

**Needs** — [`utils/BitFlags.h`](../utils/BitFlags.h.md)
**Used by** — [`network/CurlHelpers.cpp`](../network/CurlHelpers.cpp.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`scripting/ScriptEngine.cpp`](../scripting/ScriptEngine.cpp.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md) · [`scripting/bindings/MsgQueueAngelscript.cpp`](../scripting/bindings/MsgQueueAngelscript.cpp.md) · [`scripting/bindings/ScriptEventsAngelscript.cpp`](../scripting/bindings/ScriptEventsAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

Scripts subscribe to events by bit mask and receive `eventCallback(type, arg1)` or `eventCallbackEx(type, arg1, arg2, arg3, arg4, s5, s6, s7, s8)`. Engine code fires them (often asynchronously through the message queue). The numeric values are part of the **script API** and must not change.

## State

```text
RECORD ScriptEventArgs = { type; arg1; arg2ex, arg3ex, arg4ex : int; arg5ex..arg8ex : string }
```

## Events (bit n = 1 << n)

| Bit | Event | Arguments |
|---|---|---|
| 1 | EVENTBOX_ENTER | #1 type, #2 actor id, #3 node, #5 object instance, #6 box name |
| 2 | EVENTBOX_EXIT | #1 type, #2 actor id, #5 instance, #6 box |
| 3 / 4 | TRUCK_ENTER / TRUCK_EXIT | actor id |
| 5 | TRUCK_ENGINE_DIED | actor id |
| 6 | TRUCK_ENGINE_FIRE | actor id (aircraft engine failure) |
| 7 | TRUCK_TOUCHED_WATER | actor id |
| 8 – 12 | LIGHT / TIE / PARKINGBRAKE / BEACONS / CPARTICLES toggles | actor id |
| 13 / 14 | GENERIC_NEW_TRUCK / GENERIC_DELETED_TRUCK | actor id |
| 15 / 16 / 17 | TRUCK_RESET / TRUCK_TELEPORT / TRUCK_MOUSE_GRAB | actor id |
| 18 | ANGELSCRIPT_MANIPULATIONS | #1 kind: 0 console snippet, 1 script loaded (#2 unit, #3 category, #5 file), 2 load failed, 3 unloading, 4 actor sim attribute set (#2 attribute, #5 name, #6 value) |
| 19 | ANGELSCRIPT_MSGCALLBACK | #1 unit, #2 message type, #3 row, #4 col, #5 section, #6 message |
| 20 | ANGELSCRIPT_LINECALLBACK | #1 unit, #2 line, #3 call-stack size, #5 function, #6 object type, #7 object |
| 21 | ANGELSCRIPT_EXCEPTIONCALLBACK | #1 unit, #3 row, #5 function, #6 message |
| 22 | ANGELSCRIPT_THREAD_STATUS | #1 kind: 1 HTTP progress (#2 percent, #5 text), 2 success (#2 HTTP code, #3 curl code, #5 payload), 3 failure (#5 error) |
| 23 | GENERIC_MESSAGEBOX_CLICK | button |
| 24 | GENERIC_EXCEPTION_CAUGHT | #1 unit, #5 origin function, #6 type, #7 message |
| 25 | GENERIC_MODCACHE_ACTIVITY | #1 kind: entry added / modified / deleted (#2 entry, #5 file, #6 ext), bundle loaded / reloaded (#5 group) / unloaded / deleted |
| 26 | GENERIC_TRUCK_LINKING_CHANGED | #1 linked?, #2 request type, #3 master id, #4 slave id |
| 27 | GENERIC_FREEFORCES_ACTIVITY | #1 kind: added / modified / removed / deformed / broken, #2 free force id, #5/#6 stress & threshold (half-beams) |

`ALL = 0xFFFFFFFF`, `NONE = 0`.
