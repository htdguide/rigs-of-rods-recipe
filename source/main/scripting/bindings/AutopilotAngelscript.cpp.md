# source/main/scripting/bindings/AutopilotAngelscript.cpp

> Script type `AutopilotClass` and its mode enums.

**Needs** — [`Application.h`](../../Application.h.md) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [`gameplay/AutoPilot.h`](../../gameplay/AutoPilot.h.md) · [`physics/SimData.h`](../../physics/SimData.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `AutopilotClass`

`disconnect`, `get/setForceDisabled`, toggles (`toggleHeading(mode)`, `toggleAltitude(mode)`, `toggleIAS`, `toggleGPWS`), adjusters (`adjustHeading`, `adjustAltitude`, `adjustVerticalSpeed`, `adjustIAS` by delta), ILS (`getVerticalApproachDeviation`, `getHorizontalApproachDeviation`, `isILSAvailable`), and mode/value getters for heading, altitude, IAS, GPWS, vertical speed. Enums `APHeadingMode` (HEADING_NONE, HEADING_FIXED, HEADING_NAV, HEADING_WLV) and `APAltitudeMode` (ALT_NONE, ALT_FIXED, ALT_VS). See [`physics/air/AutoPilot`](../../gameplay/AutoPilot.h.md).
