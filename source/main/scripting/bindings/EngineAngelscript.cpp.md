# source/main/scripting/bindings/EngineAngelscript.cpp

> Script type `EngineClass` (the vehicle drivetrain) and gearbox enums.

**Needs** — [`AngelScriptBindings.h`](AngelScriptBindings.h.md) · [`gameplay/Engine.h`](../../gameplay/Engine.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `EngineClass`

Actor-owned reference, from `BeamClass.getEngine()`. 74 methods in three groups:

- **Definition (read-only)** — shift down/up RPM, torque, diff ratio, gear ratio(i), gear and range counts, inertia, engine type, electric/air/turbo flags, clutch force, shift/clutch/post-shift times, stall/idle RPM, idle mixture bounds, braking torque.
- **State** — accelerator, clutch, crank factor, RPM, smoke, torque, turbo psi, auto mode, gear, range, running, contact, current torque, input shaft RPM, drive ratio, engine/turbo power, idle/prime mixture, auto-shift position, accelerator needed to hold RPM, wheel spin, shift clocks, shifting flags, target gear, auto-shift behaviour and delay counters.
- **Commands** — `setAcc`, `autoSetAcc`, `setClutch`, `setRPM`, `setWheelSpin`, `setAutoMode`, `setPrime`, `setHydroPump`, `setManualClutch`, `setTCaseRatio`, `toggleContact`, `offStart`, `startEngine`, `stopEngine`, `toggleAutoMode`, `autoShiftDown`/`Set`/`Up`, `setGear`, `setGearRange`, `shift(delta)`, `shiftTo(gear)`.

Enums `autoswitch` (REAR, NEUTRAL, DRIVE, TWO, ONE, MANUALMODE) and `SimGearboxMode` (AUTO, SEMI_AUTO, MANUAL, MANUAL_STICK, MANUAL_RANGES). See [`gameplay/Engine`](../../gameplay/Engine.h.md).
