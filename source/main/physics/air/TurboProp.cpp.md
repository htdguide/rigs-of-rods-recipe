# source/main/physics/air/TurboProp.cpp

> Propeller dynamics: engine torque on blade tips, governor pitch control, blade-element lift/drag, energy correction, prop wash.

**Needs** — [`TurboProp.h`](TurboProp.h.md) · [`Actor.h`](../Actor.h.md) · [`Airfoil.h`](Airfoil.h.md) · [`gfx/GfxActor.h`](../../gfx/GfxActor.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md) · [`scripting/ScriptEngine.h`](../../scripting/ScriptEngine.h.md) · [`audio/SoundScriptManager.h`](../../audio/SoundScriptManager.h.md) · [`SimData.h`](../SimData.h.md)
**Used by** — callers of [`TurboProp.h`](TurboProp.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

The most detailed aero component; aircraft handling depends on it.

## State

See [`TurboProp.h`](TurboProp.h.md).

## `updateForces(dt, do_update)`

```text
IF do_update: air_density = ISA(ref.y); modulate sound by rpm
timer += dt
rpm = mean over blades of |v_blade − v_ref| · (60/2π) / radius
IF |mean blade position − ref| > 0.4 m: failed = true                     # prop torn apart
warm = warmup ? (timer − start)/14 (ends at 1) : 1
power = (NOT failed AND ignition) ? (0.0575 + throttle·(reverse ? 0.5 : 1)·0.9425)·full_power·warm : 0   # kW
couple = 9549.3·power / max(rpm, 10)                                      # N·m
IF torque node: apply couple/torque_dist to it, perpendicular to both the axis and the node's radial offset
tip_force = couple / radius / blades

# pitch (degrees)
IF fixed_pitch > 0: pitch = fixed_pitch
ELSE IF NOT reverse
  IF throttle < 0.01                                      # beta range
    IF pitch > 0 AND rpm < 1.4·reg: pitch −= 5·dt
    IF rpm > 1.4·reg: pitch += 5·dt
  ELSE d = clamp(rpm − reg, −5, 5); unless (d<0 AND pitch<0) or (d>0 AND pitch>45): pitch += d·dt   # constant-speed governor
ELSE                                                      # reverse
  IF rpm < 1.1·reg: pitch += 5·dt if pitch < −4 else pitch −= 5·dt
  IF rpm > 1.11·reg: pitch −= 5·dt
IF NOT failed: axis = unit(ref − back)
est_energy = 0.5·blades·m_blade·r²·ω²

FOR EACH blade
  IF NOT failed AND ignition
    span = unit(tip − ref); chord_ref = −axis × span; tip_dir = −chord_ref
    tip_total = (tip_force − rpm/10)·tip_dir                               # engine push minus friction
    FOR j IN 0..4                                                           # 5 of 6 elements; the innermost is ignored
      p = (j + 0.5)/6
      wind = −(v_tip·(1−p) + v_ref·p); w = |wind|; lift_dir = unit(span × −wind)
      chord = rotate(chord_ref, about span, pitch + twist[j] − 7)
      aoa = signed angle between chord and wind projected on the chord/normal plane
      (cz, cx) = airfoil(aoa, 1, 0)
      s = radius·blade_width/6
      F = (4·cx + cx²/(π·radius/blade_width))·0.5·ρ·w·s·wind + cz·0.5·ρ·w²·s·lift_dir
      thrust += F·axis
      ref.forces += F·p; tip_total += F·(1−p)                                # split between hub and tip
    torque += (tip_dir·tip_total)·radius
    correction = rpm > 100 ? clamp((rot_energy − est_energy)/(blades·radius·dt·ω), −1000, 1000) : 0
    tip.forces += tip_total + correction·tip_dir
  ELSE IF hub is moving                                                     # windmilling / stopped prop
    wind = −v_tip; tip.forces += ρ·((|wind|/15)/(|v_ref|/2))·wind
rot_energy += torque·dt·ω
propwash = failed ? 0 : max(0, sign·sqrt(|thrust|/(0.5·ρ·prop_area) + v²) − v)   with sign −0.1 for negative thrust
```

**Notes** — the energy term nudges the simulated rotor toward the kinetic energy implied by accumulated torque, damping numerical drift in the stiff tip-node rotation. `getRPMpc` reports `rpm/10`, i.e. percent of a nominal 1000 rpm.

## Controls

Throttle clamp 0..1; `toggleReverse` zeroes throttle and pitch; `flipStart` as for jets (warm-up 14 s); `reset` clears rpm, throttle, failure, ignition, reverse, pitch, energy. Visuals: smoke from the back node along ref→back at prop-wash speed; the "engine fire" script event fires when the failed state changes.
