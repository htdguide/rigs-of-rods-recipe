# source/main/utils/ForceFeedback.cpp

> The force-feedback law.

**Needs** — [`ForceFeedback.h`](ForceFeedback.h.md) · [`Application.h`](../Application.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`system/Console.h`](../system/Console.h.md) · [`GameContext.h`](../GameContext.h.md) · [`InputEngine.h`](InputEngine.h.md)
**Used by** — callers of [`ForceFeedback.h`](ForceFeedback.h.md) (see its Used by)
**Tier floor** — T2

## Purpose

Turns physics readings into a single signed force level on one axis.

## State

See header twin.

## `Setup`

**Contract** — takes the force-feedback device found by the [input engine](InputEngine.h.md); if none, does nothing. Otherwise disables the device's own auto-centering and sets master gain to 0 (silent until a vehicle is entered). The effect is **not** created yet: devices reject effects uploaded this early.

## `SetEnabled(on)`

**Contract** — on change, sets master gain to `io_ffb_master_gain` when enabling or 0 when disabling.

## `Update`

**Contract** — called each frame while FFB is enabled. If no device exists, prints a console warning "Disabling force feedback - no controller found" and switches `io_ffb_enabled` off. Otherwise, when the player drives a land vehicle, computes and applies the force.

```text
FUNCTION update()
  actor = player's current vehicle; IF none OR not a land vehicle: RETURN
  body = actor.ffb_body_forces()                         # inertial force at the camera
  roll  = -dot(body, actor.camera_roll_axis) / 10000     # computed, currently unused
  pitch =  dot(body, actor.camera_dir_axis)  / 10000     # computed, currently unused
  set_forces(roll, pitch, actor.wheel_speed, actor.hydro_dir_command, actor.ffb_hydro_forces())

FUNCTION set_forces(roll, pitch, wheel_speed, dir_command, stress)
  IF effect absent: create constant-force effect, 1 axis, infinite duration, level 0; upload
  level = -stress * io_ffb_stress_gain
          + dir_command * 100 * io_ffb_center_gain * wheel_speed^2    # speed-dependent centering
  effect.level = clamp(level, -10000, 10000)
  device.modify(effect)
```

**Notes** — the ±10000 range is the device API's constant-force scale. Roll and pitch are passed through for future two-axis devices and do not affect the output today.
