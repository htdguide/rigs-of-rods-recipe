# source/main/scripting/bindings/DashBoardManagerAngelscript.cpp

> Script type `DashBoardManagerClass` and enum `DashboardDataTypes`.

**Needs** — [`Application.h`](../../Application.h.md) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [`gui/DashBoardManager.h`](../../gui/DashBoardManager.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `DashBoardManagerClass`

Read/write the per-actor dashboard data slots that drive gauges: `getBool`, `getNumeric`, `getString`, `getEnabled`, `setBool`, `setInt`, `setFloat`, `setString`, `setEnabled`, `getDataType`, `getLinkIDForName`, `updateFeatures`. Enum `DashboardDataTypes`: DC_BOOL, DC_INT, DC_FLOAT, DC_STRING, DC_INVALID. Slot ids are the dashboard link ids ([`gui/DashBoardManager`](../../gui/DashBoardManager.h.md)).
