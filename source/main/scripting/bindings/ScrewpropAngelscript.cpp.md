# source/main/scripting/bindings/ScrewpropAngelscript.cpp

> Script type `ScrewpropClass`.

**Needs** — [`Application.h`](../../Application.h.md) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [`physics/water/ScrewProp.h`](../../physics/water/ScrewProp.h.md) · [`physics/SimData.h`](../../physics/SimData.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `ScrewpropClass`

Actor-owned reference. `setThrottle` / `getThrottle`, `setRudder` / `getRudder`, `getMaxPower`, `getReverse`, `toggleReverse`, `getRefNode` / `getBackNode` / `getUpNode`. See [`physics/water/ScrewProp`](../../physics/water/ScrewProp.h.md).
