# source/main/gameplay/Engine.cpp

> Engine and transmission dynamics, the automatic gearbox heuristics, turbo models, and driver input mapping.

**Needs** — [`Engine.h`](Engine.h.md) · [`AppContext.h`](../AppContext.h.md) · [`physics/ApproxMath.h`](../physics/ApproxMath.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`system/Console.h`](../system/Console.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md) · [`TorqueCurve.h`](TorqueCurve.h.md)
**Used by** — callers of [`Engine.h`](Engine.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

A one-degree-of-freedom engine (rpm) coupled to the wheels through a slipping clutch, with an automatic gearbox that imitates a driver.

## State

See [`Engine.h`](Engine.h.md).

## `UpdateEngine(dt, do_update)` — per physics step

```text
acc = max(acc, idle_mixture, prime_mixture)
  idle_mixture = rpm ≤ idle ? max_idle_mixture : min_idle_mixture
  prime_mixture (priming) = crank < 0.9 ? 1 : crank < 1 ? 10·(1 − crank) : 0     crank = 5·clamp((rpm − 1.1·idle)/(max − 1.1·idle), 0, 1)
IF do_update: injector sound ← acc
IF has_air: air_pressure += dt·rpm; above 50000 → air purge sound, reset
TURBO (see below)
T = 0
IF running AND contact: T += braking_torque · rpm/max · (1 − acc)                  # engine braking
ELSE IF NOT contact OR NOT starter: T += braking_torque
IF rpm > 100: T −= 8·hydropump_work / (rpm·0.105·dt)                               # hydraulic load
IF running AND contact AND rpm < 1.25·max: engine_torque = power(rpm)·acc; T += engine_torque
IF NOT electric AND running AND rpm < stall_rpm: stop_engine()                      # stall
IF contact AND NOT running
  electric: starter ⇒ running
  ELSE IF rpm < idle: IF starter: T += torque·e^(−2.7·rpm/idle) − braking_torque       # cranking
       ELSE running = true; engine sound
IF gear ≠ 0: T −= clutch_torque / ratio[gear]
rpm += dt·T/inertia
IF gear ≠ 0
  limit = 1.5·max(torque, power(rpm))·|ratio[1st]|
  spin  = rpm / ratio[gear]
  clutch_torque = clamp((spin − wheel_rpm)·clutch·clutch_force, ±limit) · (1 − e^(−|spin − wheel_rpm|))
ELSE clutch_torque = 0
rpm = max(rpm, 0)
IF mode < MANUAL: SHIFT SEQUENCER and AUTO CLUTCH
IF do_update AND NOT shifting AND NOT post_shifting: AUTOMATIC GEAR SELECTION
```

`power(rpm) = torque·curve(rpm) + turbo_power()`; turbo power (NEW mode only) = v1: Σ added_torque·(turbo_rpm/max_turbo_rpm); v2: turbo_psi·6.8·torque/100 ("1 psi ≈ 6 % more power").

### Shift sequencer (auto clutch modes)

```text
IF shifting
  shift_clock += dt
  IF shift_val ≠ 0
    t_decl = min(shift_time − clutch_time, clutch_time)
    IF shift_clock ≤ t_decl: r = (1 − shift_clock/t_decl)²; clutch = min(r, clutch); acc = min(r, auto_acc)   # declutch, lift throttle
    ELSE shift sound; unless autoselect is NEUTRAL: gear = clamp(gear + shift_val, −1, n); shift_val = 0
  IF shift_clock > shift_time: stop sound; acc = auto_acc; shifting = false; post_shifting = true; post_clock = 0
  ELSE IF shift_val == 0 AND gear ≠ 0 AND shift_clock ≥ shift_time − clutch_time
    acc = auto_acc/2 · sqrt((shift_clock − (shift_time − clutch_time))/clutch_time)                        # re-apply throttle
IF post_shifting
  post_clock += dt
  IF post_clock > post_shift_time: post_shifting = false
  ELSE IF auto_acc > 0: acc = auto_acc/2 · (1 + post_clock/post_shift_time)
  ELSE IF gear ≠ 0 AND wheel_rpm > rpm/ratio: clutch = max(clutch, sqrt(post_clock/post_shift_time))       # engine braking engages smoothly
```

### Auto clutch

```text
declutch_rpm = 0.75·min_rpm + 0.25·stall_rpm
IF gear == 0 OR rpm < declutch_rpm: clutch = 0
ELSE IF rpm < min_rpm (and min_rpm > declutch_rpm): clutch = min(((rpm − declutch_rpm)/(min_rpm − declutch_rpm))², clutch)
ELSE IF no pending shift AND rpm > min_rpm AND clutch < 1                           # pull-away: engage just enough
  re_torque = clamp((rpm/ratio − wheel_rpm)·clutch_force, ±1.5·power(rpm)·|ratio[1st]|) / ratio
  range = 0.4·(max − min)·sqrt(max(0.2, acc)); power_ratio = min((rpm − min)/range, 1)
  clutch = max(clutch, min(power(rpm)·min(acc, 0.9)·power_ratio, |re_torque|) / re_torque)
clutch = clamp(clutch, 0, 1)
```

### Automatic gear selection (mode AUTO, DRIVE/TWO, forward gear, not electric; once per frame)

```text
ref_wheel_rpm = (camera-forward speed of node 0) / wheel0.radius · 60/2π
# hard limits
IF (rpm > max − 100 AND gear > 1) OR wheel_rpm·ratio[gear] > max − 100
  IF (DRIVE AND gear < n AND clutch > 0.99) OR (TWO AND gear < min(2, n)): kickdown_delay = 100; shift(+1)
ELSE IF gear > 1 AND ref_wheel_rpm·ratio[gear−1] < max AND
        (rpm < min OR (rpm < min + shift_behaviour·half_range/2 AND power(lower-gear rpm) > power(current-gear rpm)))
  shift(−1)
# driver model (histories of the last 200 frames; averages over 50 and 200)
IF any of avg_acc50, avg_acc200, avg_brake50, avg_brake200 > 0.8: shift_behaviour = min(+0.01, 1)
ELSE IF everything (current and averages) < 0.5: shift_behaviour /= 1.01
candidate = gear
IF avg_acc50 > 0.8 AND rpm < max − range/3
  WHILE candidate > 1 AND wheel_rpm·ratio[candidate−1] < max − range/3 AND
        power(rpm in candidate−1)·ratio[candidate−1] > power(rpm in candidate)·ratio[candidate]: candidate −= 1       # kickdown
ELSE IF avg_acc50 > 0.6 AND acc < 0.8 AND acc > avg_acc50 + 0.1 AND rpm < min + range/2: one step down if the same test holds below min + range/2
ELSE IF avg_acc50 > 0.4 AND … (same) : one step down if it holds below min + range/3
ELSE IF gear < top (2 for TWO) AND avg_brake200 < 0.2 AND acc < min(avg_acc200 + 0.1, 1) AND rpm > avg_rpm200 − range/20
  one step up when the next gear's rpm stays above: min + range/3 (avg_acc200 in 0.4..0.6, rpm inside the middle third),
  min + range/6 (avg_acc200 in 0.2..0.4, rpm > min + range/3), min + range/6 (avg_acc200 < 0.2, rpm in min+range/6..min+range/2)
  an upshift waits until upshift_delay > 100·shift_behaviour frames; otherwise the counter resets
IF candidate < gear AND kickdown_delay > 0: candidate = gear
kickdown_delay = max(0, kickdown_delay − 1)
IF the rpm jump to candidate exceeds range/18 (down) or range/9 (up) AND |vertical g| < 0.25: shift_to(candidate)
trim histories to 200
# over-rev / wrong-way protection (AUTO and SEMI_AUTO, in gear)
IF |wheel_rpm·ratio| > 1.25·max: clutch = min(clutch, 1/(1 + (|…| − 1.25·max)/2))
IF gear·wheel_rpm < −10:         clutch = min(clutch, 1/(1 + |−10 − gear·wheel_rpm|/2))
```

(`range` = max − min rpm.) The effect: gentle driving upshifts early, hard throttle/braking learns a sportier `shift_behaviour` that holds gears longer.

### Turbo models

- **OLD** (no `engturbo`): `τ = −rpm_t/200000 + (running AND acc > 0.06 AND rpm_t < 200000 ? 1.5·acc·rpm/max : 0.1·rpm/max)`; `rpm_t += dt·τ/3e-6`. PSI = rpm_t/10000. Contributes no torque (sound/visual only).
- **NEW** per turbo: inertia `3e-6 × factor`; braking `−rpm_t/max_t`; above the operating rpm, spool `1.5·acc·(rpm − op)/(max − op)` (idle 0.1·…); the wastegate caps near its PSI and makes it *flutter* (hysteresis between ±threshold, sound), with inertia ×0.7 near the cap; without a blow-off valve, lifting off above 13 PSI adds compressor surge (τ ×3.5); anti-lag at part throttle randomly (probability 1 − chance) cuts spool and plays backfire; the blow-off valve is a second "pressure" state that follows the turbo and vents (sound) when the throttle closes above its PSI. PSI = Σ (bov or turbo rpm)/10000.

## Controls and states

- `startEngine` — `offStart`, then contact on, rpm = idle, running, gear 1 in AUTO/SEMI_AUTO, DRIVE in AUTO, ignition + engine sounds.
- `offStart` — everything off and zeroed; autoselect NEUTRAL in AUTO else MANUALMODE.
- `stopEngine` — if running: stop, fire "engine died", stop sound.
- `toggleContact` — ignition sound on/off.
- `shift(Δ)` — rejected outside −1..n; auto-clutch modes start the sequencer; manual modes shift instantly, or grind ("gearslide" sound, no change) if the clutch is engaged more than 25 %.
- `updateShifts` (after autoselect changes) — REAR → −1, NEUTRAL → 0, ONE → 1, else the lowest gear whose rpm ≤ max − 100 (capped at 2 for TWO); shift sound unless electric.
- `autoShiftUp` moves toward REAR, `autoShiftDown` toward ONE (DRIVE for electric).
- `toggleAutoMode` cycles AUTO → SEMI_AUTO → MANUAL → MANUAL_STICK → MANUAL_RANGES; entering AUTO sets DRIVE/REAR/NEUTRAL from the gear.
- `setManualClutch(v)` (manual modes only) — clutch = 1 − max(0, v).
- `setTCaseRatio(r ≥ 1)` — divides out the old ratio and multiplies in the new one on every gear.
- `getGearRatio(pos)` strips diff and transfer-case ratios. `getAccToHoldRPM = −braking_torque·(rpm/max)²/power(rpm)`.
- `pushNetworkState` overwrites rpm, acc, clutch, gear, running, contact, and mode/autoselect when given.
- `UpdateEngineAudio` — turbo, engine rpm, clutch torque and gearbox sound modulation; reverse beep while in reverse and running.

## `UpdateInputEvents(dt)` — player's actor only

```text
accel, brake from input (×0.25/0.5/0.75 when modifier keys are held)
IF NOT arcade controls OR manual gearbox: throttle = accel; brake = brake
ELSE (arcade)
  contact AND not running AND a pedal pressed → start engine
  in reverse the pedals swap roles
  when nearly stopped (|avg wheel speed| ≤ 1): brake > 0.5 with accel < 0.5 in a forward gear → reverse; the opposite → drive
AUTO: autoshift up/down keys
ignition toggle; starter while held (contact on, not running) with sound
gearbox mode toggle (console notice)
manual clutch axis (with modifiers)
sequential modes (≤ MANUAL): shift up; shift down (restricted in SEMI_AUTO arcade to forward gears); neutral; reverse; direct gear keys 1..18
H-pattern (MANUAL_STICK, MANUAL_RANGES): ranges low/mid/high/cycle only in neutral (6 gears per range);
  leaving the held gear key returns to neutral; a pressed key selects reverse, neutral, or gear (+ 6·range)
```
