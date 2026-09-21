# source/main/physics/air/TurboJet.cpp

> Turbojet spool dynamics and thrust, plus nozzle/flame/smoke visuals.

**Needs** — [`TurboJet.h`](TurboJet.h.md) · [`Application.h`](../../Application.h.md) · [`Actor.h`](../Actor.h.md) · [`SimData.h`](../SimData.h.md) · [`gfx/GfxActor.h`](../../gfx/GfxActor.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md) · [`audio/SoundScriptManager.h`](../../audio/SoundScriptManager.h.md)
**Used by** — callers of [`TurboJet.h`](TurboJet.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

A deliberately simple jet: first-order spool-up toward a throttle-dependent target, thrust proportional to spool speed.

## State

See [`TurboJet.h`](TurboJet.h.md).

## `updateForces(dt, do_update)`

```text
IF do_update: modulate engine sound by rpm %
timer += dt
axis = unit(front − back); IF |current length − ref_length| > 0.1: rpm = 0; failed = true    # engine torn off
warm = warmup ? (timer − warmup_start)/15 (ends at 1) : 1
rpm = max(rpm, 0)
torque = −rpm/100 + ((rpm < 100 AND NOT failed AND ignition) ? (0.2 + 0.8·throttle)·warm : 0)
rpm += dt · torque · 30
thrust = 0
IF NOT failed AND ignition
  thrust = max_dry · rpm/100
  afterburner = afterburnable AND throttle > 0.95 AND rpm > 80
  IF afterburner: thrust += afterburn − max_dry
ELSE afterburner = false
afterburner sound on/off
back.forces += (reverse ? −1 : +1) · thrust·1000 · axis                                    # kN → N
exhaust_velocity = thrust · 5.6 / area
```

## Controls

- `setThrottle` clamps to 0..1 (and modulates the throttle sound); `toggleReverse` only if reversible, and zeroes throttle; `flipStart` toggles ignition at most every 0.3 s, starting the warm-up and start sound when ignited and not failed.
- `reset` zeroes rpm, throttle, propwash, failure, ignition, reverse.

## Visuals

Nozzle at the back node, oriented by the axis and the ref node projected onto the plane ⟂ axis; afterburner flame length = `(ab_thrust/15)·(rpm/100)` with ±5 % random flicker; smoke along −axis at exhaust velocity, alpha and lifetime rising with throttle; a failed engine emits upward smoke.
