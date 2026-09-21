# source/main/scripting/bindings/GameScriptAngelscript.cpp

> Script type `GameScriptClass` (global `game`) and enums `ScriptCategory`, `ScriptRetCode`.

**Needs** — [`AngelScriptBindings.h`](AngelScriptBindings.h.md) · [`GameScript.h`](../GameScript.h.md) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `GameScriptClass`

Registers every method of [`GameScript`](../GameScript.h.md) under the same name, with one exception: the native `setTrucksForcedAwake` is exposed as `setTrucksForcedActive`. Functions that return arrays or dictionaries hand ownership to the script.

## Enums

- `ScriptCategory` — SCRIPT_CATEGORY_INVALID, ACTOR, TERRAIN, CUSTOM. GADGET is not exposed, so a script cannot request one by category (it loads a `.gadget` file name instead).
- `ScriptRetCode` — the engine and game codes from [`ScriptEngine.h`](../ScriptEngine.h.md#state).
