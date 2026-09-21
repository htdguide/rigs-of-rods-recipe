# source/main/scripting/bindings/InputEngineAngelscript.cpp

> Script type `InputEngineClass` (global `inputs`) and enums `inputEvents`, `keyCodes`.

**Needs** — [`utils/InputEngine.h`](../../utils/InputEngine.h.md) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `InputEngineClass`

`setEventSimulatedValue(event, value)` (a script drives a control as if a device did), `setEventStatusSupressed(event, on)` (hide an event from the game), `getEventCommand`, `getEventCommandTrimmed`, `getEventBoolValue`, `getEventBoolValueBounce(event, time)`, `isKeyDown`, `isKeyDownEffective`, `isKeyDownValueBounce`.

## Enums

- `inputEvents` — all 316 input events (EV_AIRPLANE_…, EV_BOAT_…, EV_CAMERA_…, EV_CHARACTER_…, EV_COMMANDS_01…84, EV_COMMON_…, EV_GRASS_…, EV_MAP_…, EV_MENU_…, EV_SKY_…, EV_SURVEY_MAP_…, EV_TRUCK_…, EV_ROAD_EDITOR_…) with names and values from [`utils/InputEngine.h`](../../utils/InputEngine.h.md).
- `keyCodes` — 73 keyboard codes (numbers, numpad, F1–F12, navigation, editing, modifiers), mapped to the windowing layer's key codes.
