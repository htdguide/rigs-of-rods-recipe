# source/main/scripting/bindings/ActorAngelscript.cpp

> Script type `BeamClass` (a simulated vehicle) and actor enums.

**Needs** — [`physics/Actor.h`](../../physics/Actor.h.md) · [`AngelScriptBindings.h`](AngelScriptBindings.h.md) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [`ScriptUtils.h`](../ScriptUtils.h.md) · [`physics/SimData.h`](../../physics/SimData.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `BeamClass`

Reference-counted handle to an actor; stays valid (but inert) after the actor is deleted. 81 methods:

- **Pose and motion** — `getTruckState`, `getPosition`, `getVehiclePosition`, `getRotation`, `getHeadingDirectionAngle`, `getOrientation`, `getSpeed`, `getGForces`.
- **Mass** — total, dry, loaded, initial dry/loaded; `setMass`, `setLoadedMass`, `recalculateNodeMasses`, `scaleTruck`.
- **Nodes** — count, position, initial mass, mass, velocity, forces, mass options, wheel rim/tire flags, `setNodeMass`, `setNodeMassOptions`; wheels: count, speed.
- **Shocks** — count, spring rate, damping, velocity, node 1/2.
- **Aircraft** — air-brake intensity and flaps get/set.
- **Simulation attributes** — `setSimAttribute(attr, value)` / `getSimAttribute` for tuning traction control and engine/turbo parameters at runtime (fires ANGELSCRIPT_MANIPULATIONS / ACTORSIMATTR_SET).
- **Controls** — `reset(keep position)`, parking brake / TC / ABS toggles, custom particles, `isLocked`, forced cinecam (set/clear/get), cinecam count.
- **Subsystems** — `getDashboardManager`, `getVehicleAI`, `getEngine`, aircraft engines, turbojets, turboprops, autopilot, screwprops.
- **Lights and materials** — blinker get/set, custom light visibility, beacons, brake/reverse light visibility, custom light count, flares by type, managed material instances and names.
- **Identity** — name, file name, resource group, type, section config, instance id.

Enums: `TruckState` (SIMULATED, SLEEPING, NETWORKED), `truckTypes` (NOT_DRIVEABLE, TRUCK, AIRPLANE, BOAT, MACHINE, AI), `FlareType` (12), `BlinkType` (NONE, LEFT, RIGHT, WARN), `ActorModifyRequestType` (INVALID, RELOAD, RESET_ON_INIT_POS, RESET_ON_SPOT, SOFT_RESET, RESTORE_SAVED, WAKE_UP), `ActorSimAttr` (34 tunables: TC ratio/pulse/wheelslip; engine shift RPMs, torque, diff ratio, gear ratios; engoption inertia/type/clutch/shift/clutch/post-shift times/stall/idle RPM/idle mixture/braking torque; turbo2 inertia, count, max RPM, operating RPM, BOV, wastegate, anti-lag). See [`physics/Actor`](../../physics/Actor.h.md).
