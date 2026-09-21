# source/main/physics/air/TurboProp.h

> A propeller engine (turboprop or piston): a physical rotor of 2–4 blade nodes driven by engine torque, with blade-element aerodynamics.

**Needs** — [`Application.h`](../../Application.h.md) · [`AeroEngine.h`](AeroEngine.h.md) · [`SimData.h`](../SimData.h.md)
**Used by** — [`gfx/GfxActor.cpp`](../../gfx/GfxActor.cpp.md) · [`gui/OverlayWrapper.cpp`](../../gui/OverlayWrapper.cpp.md) · [`physics/Actor.cpp`](../Actor.cpp.md) · [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`TurboProp.cpp`](TurboProp.cpp.md) · [`scripting/bindings/TurbopropAngelscript.cpp`](../../scripting/bindings/TurbopropAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

`turboprops2` and `pistonprops`. Unlike the jet, the propeller is *simulated*: its blade tips are real nodes; the engine pushes them tangentially and each blade is split into elements that produce lift and drag. Implementation: [`TurboProp.cpp`](TurboProp.cpp.md).

## State

```text
RECORD Turboprop : AeroEngine
  ref, back : node                     # hub and axis-back node; axis = unit(ref − back)
  blades[2..4] : node                  # blade tips (3rd/4th optional)
  torque_node? , torque_dist           # optional node that receives the reaction torque
  is_piston, fixed_pitch (piston: from file; turboprop: −10 ⇒ variable pitch)
  full_power : kW; max_torque = 9549.3·kW/1000
  airfoil (blade section); radius = |ref − blade0|; blade_width = 0.4; prop_area = π r²
  air_density (updated per frame), reg_speed = 1010 rpm, pitch_speed = 5 °/s, max_rev_pitch = −5
  twist[5] = {2, 6, 10, 19, 32} degrees (outer → inner)
  pitch, rpm, throttle, reverse, ignition, warmup (14 s), failed, rot_energy, propwash, indicated_torque
```
