# source/main/gameplay/CruiseControl.cpp

> Cruise control: hold speed (in gear) or rpm (in neutral) with a smoothed throttle, optional braking.

**Needs** — [`physics/Actor.h`](../physics/Actor.h.md) · [`Engine.h`](Engine.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md)
**Used by** — callers of [`physics/Actor.h`](../physics/Actor.h.md) — it implements the actor's cruise-control methods
**Tier floor** — T2


## Purpose

Methods of [`Actor`](../physics/Actor.h.md) grouped by feature. Called each frame from the actor manager's truck-feature update when cruise control is on.

## State

Actor fields `cc_mode`, `cc_target_speed`, `cc_target_rpm`, `cc_target_speed_lower_limit`, `cc_can_brake`, `cc_accs` (last 30 throttle demands).

## `cruisecontrolToggle`

On: targets = current average wheel speed and rpm. Off: targets 0, history cleared.

## `UpdateCruiseControl(dt)`

```text
disengage (toggle off) IF in a forward gear and (brake pedal > 0.05 OR target < lower limit OR parking brake)
                       OR in reverse OR engine not running OR no contact
IF in gear AND manual clutch pressed > 0.05: RETURN                       # hold still while the driver clutches
acc = engine.acc_to_hold_rpm()
IF forward gear: acc += (target_speed − wheel_speed) · total_mass/engine_power · 0.25
IF neutral:      acc += inertia · (target_rpm − rpm) / ((up − down rpm)/50)
push clamp(acc, −1, 1) into the 30-sample history
engine.auto_set_acc(clamp(mean(history), current throttle, 1))            # never lowers below the driver's pedal
accelerate key: target ×2^(dt/5) (≥ lower limit, ≤ speed limiter) or rpm ×2^(dt/5) (≤ shift-up rpm)
decelerate key: target ×0.5^(dt/5) (≥ lower limit) or rpm (≥ shift-down rpm)
readjust key: target = max(current speed, target) (≤ limiter); target rpm = current rpm
IF can_brake AND speed > target + 0.5 AND accelerator not pressed: brake = min((speed − target)·0.5, 1)
```
