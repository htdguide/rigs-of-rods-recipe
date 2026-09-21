# source/main/scripting/bindings/MsgQueueAngelscript.cpp

> Script enums `MsgType` and `FreeForceType`.

**Needs** — [`gameplay/ScriptEvents.h`](../../gameplay/ScriptEvents.h.md) · [`AngelScriptBindings.h`](AngelScriptBindings.h.md) · [`Application.h`](../../Application.h.md) · [`physics/SimData.h`](../../physics/SimData.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## Enums

- `MsgType` — every game message type (63 values, MSG_INVALID … ), same names and values as [`Application.h`](../../Application.h.md), so scripts can call `game.pushMessage(MSG_…, dict)`; which ones are accepted and their dictionary keys are listed in [`GameScript.cpp`](../GameScript.cpp.md#pushmessage).
- `FreeForceType` — FREEFORCETYPE_DUMMY, CONSTANT, TOWARDS_COORDS, TOWARDS_NODE, HALFBEAM_GENERIC, HALFBEAM_ROPE.
