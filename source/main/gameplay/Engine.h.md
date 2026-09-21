# source/main/gameplay/Engine.h

> A land-vehicle engine and gearbox: rpm integration, clutch, turbo, starter, and five transmission modes.

**Needs** — [`Application.h`](../Application.h.md) · [`utils/memory/RefCountingObject.h`](../utils/memory/RefCountingObject.h.md)
**Used by** — [`GameContext.cpp`](../GameContext.cpp.md) · [`CruiseControl.cpp`](CruiseControl.cpp.md) · [`Engine.cpp`](Engine.cpp.md) · [`VehicleAI.cpp`](VehicleAI.cpp.md) · [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`gui/panels/GUI_VehicleInfoTPanel.cpp`](../gui/panels/GUI_VehicleInfoTPanel.cpp.md) · [`network/OutGauge.cpp`](../network/OutGauge.cpp.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorForcesEuler.cpp`](../physics/ActorForcesEuler.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/Savegame.cpp`](../physics/Savegame.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`scripting/bindings/EngineAngelscript.cpp`](../scripting/bindings/EngineAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

`engine`/`engoption`/`engturbo`/`torquecurve` in the truck format become one `Engine`. The physics step reads its clutch torque to drive wheels and feeds back the wheel speed; everything else (dashboard, sound, network, AI, cruise control) reads its state. Reference-counted for scripts. Implementation: [`Engine.cpp`](Engine.cpp.md).

## State

```text
RECORD Engine
  # configuration
  shift_down_rpm (min_rpm), shift_up_rpm (max_rpm), torque (N·m), diff_ratio
  gear_ratios : [R, N, 1..n] each × diff_ratio (× transfer-case ratio); R stored negative
  inertia = 10, type 't' truck | 'c' car | 'e' electric, clutch_force (default 10000; car/electric 5000)
  shift_time 0.5 s, clutch_time 0.2 s (≤ 0.9·shift_time), post_shift_time 0.2 s
  idle_rpm = min(|min_rpm|, 800), stall_rpm = 300 (≤ 0.9·idle), max_idle_mixture 0.1, min_idle_mixture 0
  braking_torque = −torque/5 (or −engoption value), has_air (trucks), has_turbo, is_electric
  torque_curve (normalised torque vs rpm)
  turbo: mode OLD (implicit) | NEW (engturbo); version 1 (added torque) | 2 (PSI model);
         up to 4 turbos, inertia factor, max rpm (= max PSI × 10000), operating rpm, blow-off valve (min PSI 11),
         wastegate (min PSI ×10000, ± threshold), anti-lag (min rpm 3000, chance 0.9975, power 170)
  # state
  rpm, acc (throttle 0..1), clutch 0..1, clutch_torque, engine_torque, wheel_rpm (measured), ref_wheel_rpm (from speed)
  gear (−1 R, 0 N, 1..n), gear_range, running, contact (ignition), starter, priming, hydropump_work, air_pressure
  shifting / post_shifting flags + clocks, shift_val (pending relative shift), auto_acc (driver throttle during shifts)
  gearbox mode (AUTO, SEMI_AUTO, MANUAL, MANUAL_STICK, MANUAL_RANGES), autoselect (REAR, NEUTRAL, DRIVE, TWO, ONE, MANUALMODE)
  shift_behaviour 0..1 (sporty-ness learned from driver), upshift/kickdown delay counters, last-200 rpm/acc/brake histories
  turbo rpm[4], bov rpm[4], flutter flag
```

## API groups

Configuration (`SetEngineOptions`, `SetTurboOptions`, getters named after the file attributes), state getters (rpm, clutch, torque, turbo PSI, smoke, crank factor, power at rpm, acc-to-hold-rpm), controls (acc, auto acc, clutch, manual clutch, rpm, wheel spin, prime, hydro pump, transfer-case ratio), ignition (`toggleContact`, `startEngine`, `offStart`, `stopEngine`), shifting (`shift`, `shiftTo`, `setGear`, ranges, auto modes, `autoShiftUp/Down/Set`, `updateShifts`), per-step `UpdateEngine`, per-frame `UpdateEngineAudio`, `UpdateInputEvents`, `pushNetworkState`.
