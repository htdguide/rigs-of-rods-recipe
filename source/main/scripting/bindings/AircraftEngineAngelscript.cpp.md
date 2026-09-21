# source/main/scripting/bindings/AircraftEngineAngelscript.cpp

> Script type `AircraftEngineClass` and enum `AircraftEngineTypes`.

**Needs** — [`Application.h`](../../Application.h.md) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [`physics/air/AeroEngine.h`](../../physics/air/AeroEngine.h.md) · [`physics/SimData.h`](../../physics/SimData.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `AircraftEngineClass`

The common aero-engine interface (either jet or prop). `setThrottle`/`getThrottle`, `toggleReverse`/`setReverse`/`getReverse`, `flipStart`, `getRPMPercent`, `isFailed`, `getType`, `getIgnition`, `getFrontNode`, `getBackNode`, `getWarmup`. Enum `AircraftEngineTypes`: AE_UNKNOWN, AE_TURBOJET, AE_PROPELLER. See [`physics/air/AeroEngine.h`](../../physics/air/AeroEngine.h.md).
