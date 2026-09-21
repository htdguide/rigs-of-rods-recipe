# source/main/physics/Actor.cpp

> Actor lifecycle, network stream encoding/decoding, mass distribution, resets, collision escape, linking (hooks/ties/ropes), shocks2/3 and trigger laws, animator sources, lights, dashboards, and script attributes.

**Needs** — [`Actor.h`](Actor.h.md) · [`ActorManager.h`](ActorManager.h.md) · [`ActorSpawner.h`](ActorSpawner.h.md) · [`SimData.h`](SimData.h.md) · [`SlideNode.h`](SlideNode.h.md) · [`Differentials.h`](Differentials.h.md) · [`CmdKeyInertia.h`](CmdKeyInertia.h.md) · [`air/AirBrake.h`](air/AirBrake.h.md) · [`air/Airfoil.h`](air/Airfoil.h.md) · [`air/TurboJet.h`](air/TurboJet.h.md) · [`air/TurboProp.h`](air/TurboProp.h.md) · [`water/Buoyance.h`](water/Buoyance.h.md) · [`water/ScrewProp.h`](water/ScrewProp.h.md) · [`collision/Collisions.h`](collision/Collisions.h.md) · [`collision/DynamicCollisions.h`](collision/DynamicCollisions.h.md) · [`collision/PointColDetector.h`](collision/PointColDetector.h.md) · [`flex/FlexAirfoil.h`](flex/FlexAirfoil.h.md) · [`flex/FlexBody.h`](flex/FlexBody.h.md) · [`gameplay/AutoPilot.h`](../gameplay/AutoPilot.h.md) · [`gameplay/Engine.h`](../gameplay/Engine.h.md) · [`gameplay/Replay.h`](../gameplay/Replay.h.md) · [`gameplay/VehicleAI.h`](../gameplay/VehicleAI.h.md) · [`gui/DashBoardManager.h`](../gui/DashBoardManager.h.md) · [`gfx/GfxActor.h`](../gfx/GfxActor.h.md) · [`gfx/Skidmark.h`](../gfx/Skidmark.h.md) · [`network/Network.h`](../network/Network.h.md) · [`network/RoRnet.h`](../network/RoRnet.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`GameContext.h`](../GameContext.h.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`resources/tuneup_fileformat/TuneupFileFormat.h`](../resources/tuneup_fileformat/TuneupFileFormat.h.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md)
**Used by** — callers of [`Actor.h`](Actor.h.md) (see its Used by)
**Tier floor** — T2

## Purpose

Everything about an actor that is not the per-step force loop.

## State

See [`Actor.h`](Actor.h.md).

## `dispose`

**Contract** — the handler of "delete actor". Unlinks all inter-actor beams (both directions), clears hooks/ties/adjacency, stops and removes all sounds, releases autopilot, fuselage airfoil, replay, engine, AI, destroys all scene nodes/entities/flares/lights registered at spawn, the graphics actor, wings, aero engines, screw props, airbrakes, skidmarks, rail groups, collision detectors, transfer case, differentials, and the node/beam/shock/rotator/wing arrays; sets state DISPOSED. Every scene deletion is individually guarded so one failure does not leak the rest.

## Network stream

Remote actors are not simulated; they are **interpolated** from snapshots sent by their owner every **100 ms**.

**Packet** (`MSG2_STREAM_DATA_DISCARDABLE`), little-endian, must fit 8192 bytes (otherwise the program shows "Actor is too big to be sent over the net." and exits):

```text
VehicleState (RoRnet.h): time (ms, net clock), engine_speed (rpm; aero engine 0 rpm for aircraft),
    engine_force (throttle), engine_clutch, engine_gear, hydrodirstate (steering), brake,
    wheelspeed, flagmask, lightmask
node 0 absolute position: 3 × float32
nodes 1 .. first_wheel_node-1: position relative to node 0 as 3 × float16 (IEEE half) each
wheels 0 .. n-1: accumulated wheel rotation (net_rp) as float32 each
prop-animation key states: bit array, 1 bit per key, MSB first within each byte
```

Wheel nodes are **not** sent (receivers rebuild them from wheel rotation). `flagmask` bits: engine contact, engine running, gearbox mode (one of automatic/semi-auto/manual/stick/ranges), custom particles on, parking brake, TC active, ABS active, horn. `lightmask` is the full light state.

```text
FUNCTION push_network(data)                          # network receive thread
  IF size != expected_size
    once: send STREAM_REGISTER_RESULT with status -2 ("mismatch"), record the mismatch,
          print "<user> content mismatch: <file>", queue deletion of this actor
    RETURN
  split into (state, node bytes, wheel floats); unpack anim-key bits
  IF not yet initialised AND state.time > local_remote_now + 100: shift that peer's time offset forward
  append to the update queue

FUNCTION calc_network()                              # main thread, per frame
  IF fewer than 2 updates: RETURN
  rnow = max(0, net_time + time_offset[peer])
  i = last update whose time <= rnow (stop before the final one)
  t = (rnow - time_i) / (time_i+1 - time_i)
  IF t > 4: drop all updates; RETURN                 # too far behind, wait for fresh data
  IF t > 1: time_offset[peer] -= 2^t                 # we're ahead of the data: slow down
  ELSE IF i == 0 AND (queue > 5 OR (t < 0.125 AND queue > 2)): time_offset[peer] += 1   # catch up
  decode node 0 and half-float offsets for both snapshots; position = lerp by t;
      velocity = (p2 - p1) * 1000 / (time2 - time1)
  FOR EACH wheel: rotation = lerp; place its tyre (and rim) nodes on circles of wheel/rim radius
      around the two axle nodes, alternating sides, spaced 2π/(nodes/2), in the plane ⟂ axle
  update bounding boxes, average position
  engine rpm/throttle/clutch interpolated; gear, running, contact, gearbox mode from flags → engine.push_network_state
  sounds: engine rpm & injector modulation (or four aero-engine channels)
  brake, wheel speed, steering display, particles, TC/ABS indicators, parking brake, lights (setLightStateMask),
  horn start/stop, reverse beep while reverse light and engine running
  drop consumed updates (keep from i onward)
```

`sendStreamSetup` registers the local stream: type 0 (actor), status 0, time = net time, name = `"<bundle file>:<truck file>"` (bundle-qualified, ≤128 bytes), skin display name (≤60), section config (≤60).

## `recalculateNodeMasses`

**Contract** — distributes `dry_mass` over non-wheel nodes **in proportion to attached beam length**, then applies loads and minimums. Wheel (tyre) nodes keep the mass their wheel section gave them.

```text
masscount = number of non-tyre nodes with loaded mass and no override
FOR EACH non-tyre node n
  n.mass = 0                                          IF NOT loaded
         = load_mass / masscount                      IF loaded, not overridden
         = override_loadweight[n]                     IF loaded and overridden
total_len = Σ over non-virtual beams of (refL/2 for each end that is not a tyre node)
FOR EACH non-virtual beam b
  half = b.refL * dry_mass / total_len / 2
  add half to each end that is not a tyre node
FOR EACH rope: rope end node mass = 100 kg
FOR EACH cinecam: its node mass = the cinecam's node_mass from the file (default 20 kg)
FOR EACH non-tyre node (optionally skipping loaded nodes when 'minimass l')
  n.mass = max(n.mass, minimass[n])                    # default 50 kg
total_mass = Σ node masses; log "TOTAL VEHICLE MASS: <kg> kg"
```

With `diag_truck_mass` on, each stage is logged.

## Resets

- **`reset(keep_position)`** — queues RESET_ON_SPOT or RESET_ON_INIT_POS.
- **`SyncReset(reset_position)`** — fires "truck reset"; zeroes g-forces, controls, blinkers, brakes, speeds, cruise control, origin; unlinks all inter-actor beams; puts nodes back to their initial positions with zero velocity/force; restores every beam (yield stresses = default deform, strength = initial strength, L = refL, not broken/disabled), re-applies node-beam scales, disables hook and tie beams, clears ropable counters, resets wheels (speed, torque, detached), restarts the engine if `sim_spawn_running`, clears differential windup, aero engines, screw props, rotator angles, wing damage, autopilot, buoyancy sinking, hydro inertia, flexbodies, command key values and one-press states, slide nodes (and unlocks them). If `reset_position` is false the actor is then re-oriented to its current heading at its current position and lifted if below ground or water. Sets `ongoing_reset`.
- **`SoftReset`** — fires "truck reset"; only lifts the actor (and linked actors) so no node is below the ground/water beneath it; sets `ongoing_reset`.
- **`softRespawn(pos, rot)`** — hard reset without moving, then places every node at `pos + rot * spawn_offset` (the tuned spawn layout) with zero motion.
- **`resetPosition(x, z, set_init, miny)`** — translate horizontally so node 0 is at (x, z); raise so the lowest node is at `miny` (and above static water); raise further until no ground-contact node is below terrain; then raise in 1 mm steps (up to 1 m) until the first contacting node is clear of collision meshes; optionally store as initial positions.
- **`HandleInputEvents`** (during an ongoing reset / live repair) — applies accumulated rotation requests about a centre (angle snap rounds heading to a multiple of the requested division), rotating positions, velocities and forces, and translation requests.

## Collision escape

**Contract** — `calculateCollisionOffset(dir)` walks the actor along `dir` in 5 cm steps (up to `|dir|`) until it no longer: has contactable nodes near another actor's cab triangles (point detector, range 3× collision range), has contactable nodes within `max(0.05, sqrt(max(radius_a, radius_b))/50)` of another actor's contactable nodes, has cab triangles near other actors' contacters, or has contactable beams piercing another actor's cabs (ray–triangle test). `resolveCollisions(max, consider_up)` tries left, then right (preferring left unless right is ≥10 % shorter), front/back likewise, picks lateral unless sagittal is ≥20 % shorter, optionally up if ≥20 % shorter, adds 20 cm horizontal margin, and moves the actor there. Used when spawning into occupied space.

## `UpdatePhysicsOrigin`

**Contract** — when node 0 is more than 100 m from the origin, move the origin to node 0's position and subtract that offset from every node's relative position.

## Shock laws

### `CalcShocks2(beam, delta, &k, &d, v)` — progressive shocks

```text
IF v > 0 (extending): k = springout; d = dampout; f = min((delta / (longbound·L))², 1) if longbound ≠ 0 else 1
                      k += sprogout·k·f; d += dprogout·d·f
ELSE (compressing):   k = springin;  d = dampin;  f = min((delta / (shortbound·L))², 1) if shortbound ≠ 0 else 1
                      k += sprogin·k·f;  d += dprogin·d·f
IF soft bump bounds ('s')
  pre = 0.8·L
  IF delta > longbound·pre: recompute extension progression, then add
        k += (k+100)·sprogout·min(((delta - longbound·pre)·5 / (longbound·L))², 1)   (same for d with dprogout)
        and if compressing (v < 0) use plain springin/dampin
  ELSE IF delta < -shortbound·pre: mirror image (note: uses sprogout/dprogout for the bump term — original)
        and if extending use plain springout/dampout
  IF beyond either bound: k = max(k, sbd_spring); d = max(d, sbd_damp)
ELSE IF beyond either bound: k = sbd_spring; d = sbd_damp              # hard bump stop
```

### `CalcShocks3(beam, delta, &k, &d, v)` — two-stage (slow/fast) damping

```text
IF delta > longbound·L:  over = delta - longbound·L; k += (sbd_spring - k)·over; d += (sbd_damp - d)·over
ELSE IF delta < -shortbound·L: over = -delta - shortbound·L; same blend
ELSE IF v > 0: s = clamp(|v|, 0.15, 20); k = springout
               d = (dampout·dslowout·min(s, splitout) + dampout·dfastout·max(0, s - splitout)) / s
ELSE IF v < 0: same with springin, dampin, dslowin, dfastin, splitin
```

## `CalcTriggers(beam, delta, trigger_hooks)`

A trigger is a free-moving beam that sends command-key input (or hook/engine actions) when stretched past `longbound` or compressed past `shortbound`. `cmdshort` / `cmdlong` are the command keys (or counts, or engine parameters) configured in the file.

```text
IF NOT trigger or disabled: RETURN
IF outside bounds
  switch_state = max(0, switch_state - DT)
  CASE kind OF
    blocker:          disable the next cmdshort trigger beams (beam indices i+1 .. i+cmdshort)
    inverted blocker: enable the next cmdlong trigger beams
    command blocker:  release the block on command key cmdshort
    command switch:   if not already switched in this excursion: swap cmdshort/cmdlong of every other trigger
                      that uses the same pair; switch_state = boundary_timer
    plain trigger, past longbound:
      hook unlock/lock (on the frame's last step): queue HOOK_UNLOCK/LOCK for hook group cmdlong
      engine trigger: engine action cmdlong with value 1
      else if key cmdlong not blocked: continuous → key cmdshort input = 1; else key cmdlong input = 1
    plain trigger, past shortbound:
      hook: group cmdshort
      engine: action cmdlong, value 0 if continuous else 1
      else if key cmdshort not blocked: key cmdshort input = 0 if continuous else 1
ELSE (inside bounds)
  continuous: value = clamp((delta/L - shortbound) / (longbound - shortbound), 0, 1)
              engine trigger → engine action; else both keys' trigger input = value
  blocker: re-enable the next cmdlong triggers
  inverted blocker: disable the next cmdshort triggers
  command switch that was switched: reset switch_state
  command blocker: block key cmdshort
```

Engine trigger actions (`engineTriggerHelper`): 0 clutch := value, 1 brake := value, 2 throttle := value, 3 target rpm (not implemented), 4 shift up, 5 shift down.

## `CalcAnimators(hydro, &c, &n)` — sources for animator beams

Each enabled source overwrites or subtracts from the control value `c` and increments `n` (the divisor):

| Source | Value |
|---|---|
| boat rudder / throttle | average of all screw props' rudder / throttle |
| difflock | wheel diff 0: Open 0, Split 0.5, Locked 1 (0.5 if none) |
| heading | heading_deg / 360 |
| torque | while engine crank factor rises: `c -= crank/10`, else 0; clamp ≥ −1 |
| shifter (param 3: sequential) | ±1 for 0.2 s after an up/down shift |
| shifter (param 1: H left/right) | neutral −0.5, reverse 1, else `c -= floor((gear-1)/2)` |
| shifter (param 2: H up/down) | 0.5 neutral, 1 reverse, else gear parity |
| shifter (param 4: linear) | `c -= (gear+2)/(num_gears+2)` |
| parking brake | `c -= parking_brake` |
| speedo | `c -= 3·wheel_speed/speedo_max_kph` |
| tacho | `c -= rpm/shift_up_rpm` |
| turbo | `c -= psi·3.34/67` |
| brake / accel / clutch | `c -= brake`; `c -= throttle + 0.06`; `c -= |1 − clutch|` |
| aero rpm (engine = param) | percent → dial angle (−5 + 1.9167·p below 60 %, 110 + 4.075·(p−60) below 110 %, else 314), `c -= angle/314` |
| aero throttle / torque / pitch / status | `c -= throttle`; turboprop torque% /120; pitch/120; 0 off, 0.5 on, 1 failed |
| airspeed | indicated knots/100 (tropospheric density correction) |
| vertical speed | `c -= vs_ft_min/6000`, clamp ±1 |
| altimeter (param 1/2/3) | 100 k-ft (limited), 10 k-ft and 1 k-ft (fractional, wrapping) needles |
| angle of attack | wing 4 AoA/25 (0 below 10 kt), clamp ±1 |
| roll | roll_deg/180 with upside-down wrap into ±1 |
| pitch | pitch_deg/90 |
| airbrake | `c -= intensity/5` |
| flaps | `c = FLAP_ANGLES[flap]` |

## Linking: hooks, ties, ropes

Linking changes are only performed on the main thread (from linking-request messages).

**`AddInterActorBeam` / `RemoveInterActorBeam`** — maintain the actor's inter-beam list and the manager's global `beam → (actor A, actor B)` table. When the *direct* link status of the pair changes, recompute the linked-actor sets of both actors and everything linked to them, propagate (on link) or reset (on unlink) physics-paused and debug-view states across the group, and fire "truck linking changed (linked?, type, a, b)".

**`hookToggle(group, mode, mouse_node, unlock_filter)`**

```text
FOR EACH hook h
  filters:  MOUSE_TOGGLE → only the hook on mouse_node
            TOGGLE with group -1 → hooks with group ≥ -1 (manual "lock" key)
            LOCK/UNLOCK with group -2 → only auto-lock hooks with group ≤ -2
            LOCK/UNLOCK with group ≤ -3 → only hooks of exactly that group (trigger-driven)
            LOCK/UNLOCK with group ≥ -1 → nothing
            LOCK while h.timer > 0 → skip (re-lock delay)
            RESET with unlock_filter → only hooks locked to that actor
  IF mode is not UNLOCK/RESET AND h is UNLOCKED
    search every non-sleeping actor (self only if selflock): nearest node within h.lockrange,
      skipping lockgroup-9999 nodes, the hook node itself, and (if h has a lockgroup) nodes of other lockgroups
    found → h.lock_node, h.locked_actor, state PRELOCK; if its beam was disabled: attach it to the node,
      L = current distance, enable, register as inter-actor beam
  ELSE IF h is LOCKED/PRELOCK AND (mode != LOCK OR the beam is no longer inter-actor)
    UNLOCKED; unregister; auto-lock hooks restart their re-lock timer (0 on RESET);
    beam reattached to node 0 with L = distance, disabled
```

**`tieToggle(group, mode, unlock_filter)`** — first unties every tied tie of the group (disables its beam, points it back at node 0, unregisters). If none was tied and mode is TOGGLE, each untied tie searches all non-sleeping actors (not self if no-self-lock) for the nearest ropable within its reference length that is free (or multi-lock) and is not the tie's own node; ties to it, sets L = refL, starts *tying*, and registers the inter-actor beam. Fires "tie toggle".

**`ropeToggle(group, mode, unlock_filter)`** — locked/prelocked ropes unlock (RESET or TOGGLE); unlocked ropes on TOGGLE lock to the nearest free ropable within the rope's current length (any actor, including self). The rope's end node is pinned to the ropable each step (see [`CalcRopes`](ActorForcesEuler.cpp.md#calcropes)).

**`DisjoinInterActorBeams`** — reset all own hooks/ropes/ties, and ask every locally simulated actor to reset any of its hooks/ties/ropes locked to this actor.

`isLocked` = any hook LOCKED; `isTied` = any tie tied.

`DetermineLinkedActors` — breadth-first over the manager's link table starting from this actor; the result includes indirect links.

## Lights

- `setBlinkType(type)` sets exactly one of left/right/warn light bits (or none) and starts/stops the turn-signal sound; `toggleBlinkType` turns off if already set.
- `autoBlinkReset` — once steering passes `io_blink_lock_range` in the blink direction, returning past it cancels the blinker.
- `toggleHeadlights` — when the player's actor has `forwardcommands`, also toggles every other local actor with `importcommands`; flips the headlight bit, syncs cab lights, fires "light toggle".
- `setLightStateMask(mask)` — toggles headlights/beacons via their functions when they differ, sets blink type, then stores the mask.
- `importLightStateMask(mask)` — for linked actors: bits in `flaregroups_no_import` keep their local value.
- `updateFlareStates(dt)` — per flare: blink timer toggling every `blinkdelay`; visibility by type (headlight/tail ← headlight bit; high beam, fog, side, brake, reverse bits; blinkers ← own bit or warn; dashboard ← dashboard value; user ← custom light n); `intensity` = visibility, or a smoothed ramp for `flares3`.
- custom lights 0..9 map to lightmask CUSTOM1..10; out-of-range ids are logged.
- `beaconsToggle` (only if flares are enabled) flips beacons and fires "beacons toggle"; `parkingbrakeToggle` flips and plays park sound, fires event; TC/ABS toggles respect `notoggle`.

## `updateVisual(dt)`

Blinker auto-reset; sound sources follow their nodes (position, velocity), airspeed (knots) and wheel speed (km/h) sound modulation; aircraft play random radio chatter every 11–30 s; control surfaces: combine autopilot and pilot aileron/rudder/elevator (clamped ±1) and set each wing's deflection by its surface letter (a: aileron, b: −aileron, r: rudder, e/S/T: elevator, f: flaps, c/V: (ail+elev)/2, d/U: (−ail+elev)/2, g: (ail+flap)/2, h: (−ail+flap)/2, i: (−elev+rud)/2, j: (elev+rud)/2) and update wing physics vertices; set hydro aileron/rudder/elevator commands.

## `updateDashBoards(dt)`

Publishes values to the dashboard: gear, gear count, gear string (`"g/n"`, `N`, `R`), automatic gear string with colour codes, auto-gear, clutch, throttle, rpm, running, turbo (psi·3.34), ignition/battery (contact without running), clutch warning (|torque| ≥ 10·clutch force), brake, speed km/h and mph (smoothed 0.3 new/0.7 old), roll and pitch degrees, stabiliser correction, parking brake, hook locked, low hydraulic pressure, TC/ABS mode (1 off, 2 on, 3 active), tie mode, screw prop throttle/rudder, water depth and speed (knots), aero-engine throttle/failed/rpm%, wing AoA, indicated airspeed, altitude and altitude string (hundreds of feet), odometers, all light bits and turn-signal lamps. On first update it enables/disables dashboard features according to what the vehicle has.

## Misc

- `scaleTruck(f)` — scale damping, lengths, hydro lengths and speeds, node positions about node 0, velocities, forces and masses by `f` (spring constants and stresses are intentionally not scaled — they describe material).
- `getRotation` = `atan2(dir·X, dir·(−Z))`; `getDirection` = main camera direction corrected by `camera_dir_corr`; `getOrientation` builds a basis from camera direction and roll.
- `calculateAveragePosition` — custom camera node, else cinecam 0 (extern mode CINECAM), else a chosen node (mode NODE), else the mean of all nodes.
- `UpdateBoundingBoxes` — see state; 5 cm padding; event-box box only includes nodes within 15 m of the main camera node.
- `mouseMove(node, pos, force)` — grab force scaled by `(total_mass/3000)^0.75`.
- `calculateLocalGForces` — camera-frame accelerations (vertical including gravity, longitudinal, lateral) divided by g; maxima recorded after 0.5 s.
- `searchBeamDefaults` — random search of node-beam scale factors: run a skip phase, then measure 500 steps of velocity/stress/movement/breakage; keep the new scales only if no metric exceeds the reference.
- `setSimAttribute` — refused in multiplayer; logs and fires an "AngelScript manipulation" event; sets the named engine/TC/turbo parameter (gear-ratio array not implemented).
- `ensureWorkingTuneupDef` creates `"Tuned <file>"`; `WriteDiagnosticDump` writes node and beam tables to the logs folder.
- `UpdatePropAnimInputEvents` — for each event-driven prop animation: toggle on press if `eventlock`, else follow the event.
