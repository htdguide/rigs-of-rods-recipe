# source/main/scripting/bindings/TurbojetAngelscript.cpp

> Script type `TurbojetClass`.

**Needs** — [`Application.h`](../../Application.h.md) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [`physics/air/TurboJet.h`](../../physics/air/TurboJet.h.md) · [`physics/SimData.h`](../../physics/SimData.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `TurbojetClass`

Reference type, lifetime owned by the actor. Read-only: `getMaxDryThrust`, `getAfterburner` (on/off), `getAfterburnerThrust`, `getExhaustVelocity`. Obtained from `BeamClass.getTurbojet(i)`; see [`physics/air/TurboJet`](../../physics/air/TurboJet.h.md).
