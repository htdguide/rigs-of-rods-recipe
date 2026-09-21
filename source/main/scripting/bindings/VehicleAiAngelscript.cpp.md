# source/main/scripting/bindings/VehicleAiAngelscript.cpp

> Script type `VehicleAIClass` and enums `aiEvents`, `AiValues`.

**Needs** — [`gameplay/VehicleAI.h`](../../gameplay/VehicleAI.h.md) · [`AngelScriptBindings.h`](AngelScriptBindings.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `VehicleAIClass`

`setActive`, `isActive`, `addWaypoint(id, position)`, `addWaypoints(dictionary)`, `addEvent(waypoint id, event)`, `setValueAtWaypoint(waypoint id, value type, value)`, `getTranslation(offset, waypoint)`. Enums `aiEvents` (AI_LIGHTSTOGGLE, AI_WAIT_SECONDS, AI_BEACONSTOGGLE) and `AiValues` (AI_SPEED, AI_POWER). See [`gameplay/VehicleAI`](../../gameplay/VehicleAI.h.md).
