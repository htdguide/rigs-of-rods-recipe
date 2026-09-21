# source/main/gfx/GfxActor.cpp

> Snapshotting actor state, and turning it into props, instrument animations, flares, beacons, particles, cameras and meshes.

**Needs** — [`GfxActor.h`](GfxActor.h.md) · [`physics/ApproxMath.h`](../physics/ApproxMath.h.md) · [`physics/air/AirBrake.h`](../physics/air/AirBrake.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/water/Buoyance.h`](../physics/water/Buoyance.h.md) · [`physics/collision/Collisions.h`](../physics/collision/Collisions.h.md) · [`gui/DashBoardManager.h`](../gui/DashBoardManager.h.md) · [`DustPool.h`](DustPool.h.md) · [`gameplay/Engine.h`](../gameplay/Engine.h.md) · [`GameContext.h`](../GameContext.h.md) · [`GfxScene.h`](GfxScene.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`gui/GUIUtils.h`](../gui/GUIUtils.h.md) · [`HydraxWater.h`](HydraxWater.h.md) · [`physics/flex/FlexAirfoil.h`](../physics/flex/FlexAirfoil.h.md) · [`physics/flex/FlexBody.h`](../physics/flex/FlexBody.h.md) · [`physics/flex/FlexMeshWheel.h`](../physics/flex/FlexMeshWheel.h.md) · [`physics/flex/FlexObj.h`](../physics/flex/FlexObj.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md) · [`utils/MeshObject.h`](../utils/MeshObject.h.md) · [`MovableText.h`](MovableText.h.md) · [Seam: Immediate-mode GUI](../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui) · [`network/RoRnet.h`](../network/RoRnet.h.md) · [`physics/ActorSpawner.h`](../physics/ActorSpawner.h.md) · [`physics/SlideNode.h`](../physics/SlideNode.h.md) · [`SkyManager.h`](SkyManager.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`physics/air/TurboJet.h`](../physics/air/TurboJet.h.md) · [`physics/air/TurboProp.h`](../physics/air/TurboProp.h.md) · [`utils/Utils.h`](../utils/Utils.h.md)
**Used by** — callers of [`GfxActor.h`](GfxActor.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

The per-actor render update. Most math here is *placement relative to three nodes*: a thing attached to nodes (ref, x, y) with offset (ox, oy, oz) sits at `ref + ox·(x − ref) + oy·(y − ref) + oz·unit((y − ref) × (x − ref))`, oriented by the basis (unit(x − ref), normal, unit(x − ref) × normal). Props, flares, airbrakes, the driver seat and flexbody centres all use this rule.

## State

See [`GfxActor.h`](GfxActor.h.md).

## `UpdateSimDataBuffer` (while physics is synchronised)

Copies into `ActorSB`: state, pause, cinecam, net name/colour, type, rotation centre, node-0 velocity, heading, direction, wheel/top speed, bounding box, camera-0 nodes; every node position and `has_contact = ground or mesh contact`, `is_wet` from the node visuals; for every rod its current end nodes (hooks and ties may retarget), target actor for inter-actor beams, visible = not disabled and not broken; airbrake ratios; command key values 1..84; prop-animation key states; aero engines (turboprop torque % = 100·indicated/max, pitch; jet afterburner, thrust, exhaust velocity); steering, brake; engine data (gear, autoshift, rpm, turbo psi, throttle, current torque, input shaft rpm, drive ratio, clutch, gears, max rpm, smoke); active wheel-diff type; tyre pressure; light mask, smoke, parking brake, custom particles; aero control states, flaps, airbrake, wing 4 AoA; autopilot state; gui settings.

**Notes** — `simbuf_engine_crankfactor` and the screw-prop list are never filled, so `torque` and boat rudder/throttle prop animations read 0 in this snapshot. Recorded as observed; a rebuild that fills them changes those instruments' behaviour.

## `CalcPropAnimation(anim, cstate, div)`

Each enabled source changes `cstate` (most *subtract*, matching the animator conventions of the physics side) and increments `div`:

| Source | Contribution |
|---|---|
| boat rudder / throttle | mean screw-prop value (assigned) |
| difflock | Open 0, Split 0.5, Locked 1; no diffs 0.5 (assigned) |
| heading | heading°/360 (assigned) |
| torque | `−crank/10` while rising, else 0; ≥ −1 |
| gear n | +1 when the engine is in gear n (R = −1, N = 0) |
| sequential shifter | ±1 for 0.2 s after a shift, or from command keys named by the limits; smoothed |
| H left/right | N −0.5, R 1, else −floor((g−1)/2); smoothed |
| H up/down | 0.5, R 1, else g mod 2; smoothed |
| linear / auto linear | −(g+2)/(n+2) (auto: forward gears count as 1, n = 1); smoothed |
| parking brake, brake | −value |
| speedo | −3·speed/max_kph |
| tacho | −rpm/max rpm |
| turbo | −psi·3.34/67 |
| accel | −(throttle + 0.06) |
| clutch | −|1 − clutch| |
| signal stalk | −(−1 left / +1 right) |
| aero rpm | −dial/314, dial = −5 + 1.9167·p (p < 60 %), 110 + 4.075·(p − 60) (< 110 %), 314 |
| aero throttle | −throttle |
| turboprop torque / pitch | value/120 (assigned) |
| aero status | 0 off, 0.5 on, 1 failed (assigned) |
| airspeed | −IAS(kt)/100 (ISA density) |
| VVI | −(v_y·196.85)/6000, clamped ±1 |
| altimeter 3 / 2 / 1 | −frac(h·1.1811/360), −frac(h·1.1811/3600) (≥ −1), −h·1.1811/36000 (≥ −1) |
| AoA | −aoa(wing 4)/25 (0 below 10 kt), ±1 |
| roll | roll°/180 with inverted wrap (assigned) |
| pitch | pitch°/90 (assigned) |
| airbrake | −intensity/5 |
| flaps | FLAP_ANGLES[flap] (assigned) |

*Smoothing* (`UpdateSmoothShift`) moves toward a new target in steps of `(dt/shifter_anim_time)·Δ`, clamping at the target.

## `UpdatePropAnimations(dt)`

```text
FOR EACH prop: rx = ry = rz = 0
  FOR EACH animation
    cstate from sources; + event key state (keys consumed in order); + dashboard input value; + steering, aileron,
    elevator, aero rudder states; + 1 if permanent
    cstate ×= ratio; ×= opt5 if set (bounce direction)
    rotation modes:
      auto-animate: rota[axis] += cstate·dt·2000 (frame-rate normalised); limit check on that axis
      else: r[axis] += cstate
      beyond upper (lower) limit: no-flip → clamp and reverse bounce; else wrap to the other limit
    offset modes: offset = original + cstate; auto-animate integrates the original with the same limit rules
  pp_rot = Z(rz + rota.z)·Y(ry + rota.y)·X(rx + rota.x)
```

## `UpdateProps(dt, is_player)`

Propeller blade vs spinner meshes swap at 200 rpm. Other props are hidden unless their camera mode is "always" or matches the current cinecam (hidden props are not placed). Placement by the three-node rule with `pp_rot`. Steering wheels: pivot at `position + orientation·wheel_pos`, orientation `· X(−59°) · Y(steering·wheel_rot_degree)`. When the beacon bit changes, enable/disable beacons; with flares enabled and beacons on, update each beacon.

**Beacons** (`UpdateBeaconFlare`; hidden beyond 100 m from the camera): `b` custom — a spot light rotating at its rate about the prop's local Z, the flare billboard 0.1 m toward the camera sized `a³` where a = light direction · view direction (invisible when facing away); `p` lightbar — four such lights at x = −0.64, −0.32, +0.32, +0.64 (0.14 up), blue/blue/red/red; `r` red — flashes once per unit of accumulated angle; `L`/`R` wing navigation lights (static billboards); `w` wingtip strobes (flash like `r`). Light sources are suppressed for non-player actors in "current vehicle only" flare mode.

## `UpdateFlares(dt, is_player)`

Per flare: material flare glow on when intensity > 0.3; billboard visible when intensity > 0; incandescent (`flares3`) alpha = intensity; light source visible if intensity > 0 and allowed by the flare mode (current vehicle headlights only / all headlights / all lights). Placement by the three-node rule; hidden beyond 500 m; amplitude = normal · view direction (negative size → always full); billboard `amplitude·size`; light 0.2 m in front, pointing `−normal − (0, 0.2, 0)`.

## Particles (`UpdateParticles`)

For wettable nodes: leaving water starts a 5 s drip (and vapour for hot nodes). Submerged nodes within 0.2 m of the surface moving faster than 2 m/s splash and ripple. Ground contact by fx type: dusty → dust in the ground colour; clumpy → clumps when faster than 1 m/s; hard → tyre nodes: screech sound when `min(slip, avg slip) > 5` (modulated by excess/5), tyre smoke when avg slip > 8; other nodes (unless no-sparks): sparks when avg slip > 5 and current slip > 5.

## Other updates

- Rods: stretched unit beam mesh between the two snapshot positions (target actor's snapshot for inter-actor rods), scale (diameter, length, diameter), rotated from +Y.
- Wheels and flexbodies: queue `flexitCompute` / `computeFlexbody` on the thread pool (visible flexbodies only), later join and upload.
- Airbrakes: three-node placement, rotated by `−ratio·max_angle` about the x axis.
- Custom particles / exhausts: follow emitter nodes, direction from the direction node; exhaust smoke alpha `0.02 + smoke·0.06`, lifetime ×25, speed `1 + 2·smoke … 2 + 3·smoke`, off when the engine does not run.
- Video cameras: mirrors reflect the view direction about the camera plane; video cams orient by (−x, −y, −normal) × user rotation; tracking cams look at their target node; mirror props place a camera at the prop (±0.22 m sideways) mirroring the main camera across the mirror plane, rolled by the actor pitch. Only updated when online.
- Driver position: three-node rule on the seat prop, rotated by its prop rotation and 180° about Y.
- Net labels above remote (or own, unless hidden) actors in multiplayer: at bounding-box top + distance/100.
- Debug views (ImGui overlays): skeleton/nodes/beams (ids, masses, strength and stress), wheels (id, rpm, torque), shocks (length, spring, damping, velocity), rotators (ids, angle, error), slide nodes, submesh (cab triangles), buoyancy; cycling skips views the actor cannot show.
- Cab lights: swap the cab material between the emissive and plain templates (made once when the cab material has an emissive pass).
