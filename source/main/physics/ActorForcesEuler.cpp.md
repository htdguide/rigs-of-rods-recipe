# source/main/physics/ActorForcesEuler.cpp

> One 0.5 ms physics step of one actor: integrate nodes, then accumulate every force source — aero, buoyancy, drivetrain, wheels, actuators, beams — for the next step.

**Needs** — [`Actor.h`](Actor.h.md) · [`ActorManager.h`](ActorManager.h.md) · [`ApproxMath.h`](ApproxMath.h.md) · [`SimData.h`](SimData.h.md) · [`Differentials.h`](Differentials.h.md) · [`CmdKeyInertia.h`](CmdKeyInertia.h.md) · [`air/AeroEngine.h`](air/AeroEngine.h.md) · [`air/AirBrake.h`](air/AirBrake.h.md) · [`air/Airfoil.h`](air/Airfoil.h.md) · [`flex/FlexAirfoil.h`](flex/FlexAirfoil.h.md) · [`water/Buoyance.h`](water/Buoyance.h.md) · [`water/ScrewProp.h`](water/ScrewProp.h.md) · [`collision/Collisions.h`](collision/Collisions.h.md) · [`gameplay/Engine.h`](../gameplay/Engine.h.md) · [`gameplay/Replay.h`](../gameplay/Replay.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`gfx/GfxWater.h`](../gfx/GfxWater.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md) · [`GameContext.h`](../GameContext.h.md) · [`system/Console.h`](../system/Console.h.md)
**Used by** — callers of [`Actor.h`](Actor.h.md) — it implements the actor's per-step force calculation
**Tier floor** — T2: the hot loop of the whole program

## Purpose

This is the soft-body simulation. Each actor is a set of point masses (nodes) joined by damped springs (beams). Every 0.5 ms the actor integrates node motion and recomputes all forces. The model is **symplectic (semi-implicit) Euler**: velocity is updated from the force accumulated in the *previous* step, then position from the new velocity; after integration the force accumulator is reset to gravity and every subsystem adds to it. The order of subsystems below is part of the behaviour.

The [actor manager](ActorManager.cpp.md) drives it: per frame it runs N steps; within each step it calls `CalcForcesEulerPrepare` for every actor, resolves inter-actor collisions and inter-actor beams, then `CalcForcesEulerCompute` for every actor in parallel. `doUpdate` is true on the last step of a frame and gates work that only matters once per frame (sounds, debug values, UI).

## State

Operates on the actor's arrays (see [`Actor.h`](Actor.h.md)) — nodes, beams, shocks, wheels, hydros, command keys, rotators, ties, hooks, ropes — plus the terrain (gravity, water, collisions).

## `CalcForcesEulerPrepare(doUpdate)`

**Contract** — returns false (actor skipped this step) while a reset is in progress, when the actor's physics is paused, or when it is not locally simulated. On the last step of a frame it queues an "auto-lock hooks" linking request (hook group −2) for the main thread. Then advances hooks and locked ropes (below).

## `CalcForcesEulerCompute(doUpdate, num_steps)`

```text
FUNCTION compute(doUpdate, num_steps)
  calc_nodes()              # integrate; must run right after inter-actor collisions
  update_bounding_boxes()
  calc_event_boxes()
  calc_replay()
  calc_aircraft_forces()    # airbrakes, aero engines, screw props, wings
  calc_fuse_drag()
  calc_buoyance()
  calc_differentials()
  calc_wheels()
  calc_shocks()             # active stabilisers
  calc_hydros()
  calc_commands()           # commands, rotators, hydraulic pump load
  calc_ties()
  calc_engine()             # after commands/engine triggers updated
  calc_mouse()
  calc_beams()              # the spring network + deformation + breaking
  calc_cab_collisions()     # self/inter collisions of node vs. cab triangles
  update_slide_node_forces()
  calc_force_feedback()
```

## `CalcNodes`

Integration, gravity, drag and water forces for every node.


```text
FUNCTION calc_nodes()
  g = terrain.gravity; water = terrain.water
  water_contact = false
  FOR EACH node n
    IF NOT n.no_ground_contact
      before = n.abs
      contact = collisions.ground(n, DT) OR collisions.node_vs_static_objects(n, DT)   # may move n.abs and change n.velocity
      n.has_ground_contact = contact
      IF contact OR n.has_mesh_contact
        last_fuzzy_ground_model = n.last_collision_gm
        n.rel += n.abs - before        # keep relative position consistent with the collision correction
    IF n is the main camera node: camera_gforces_accu += n.forces / n.mass
    IF NOT n.immovable
      n.velocity += n.forces / n.mass * DT
      n.rel      += n.velocity * DT
      n.abs       = origin + n.rel
    # start next step's accumulator
    n.forces = (0, n.mass * g, 0)
    speed ≈ |n.velocity|
    IF speed > 6860 m/s (Mach 20) AND no reset pending
      queue RESET_ON_SPOT for this actor; reset_pending = true        # "explosion" guard
    IF actor has a fuselage airfoil: n.forces += fuse_drag            # replaces per-node drag
    ELSE IF turbulent drag not disabled
      n.forces += -DEFAULT_DRAG * speed * n.velocity                  # quadratic air drag
      n.forces += (DEFAULT_DRAG * speed² * 0.005) * random_vector(-1..1)   # turbulence noise
    IF water AND water.is_under_water(n.abs)
      water_contact = true
      IF actor has no buoyant cab triangles
        n.forces += -DEFAULT_WATERDRAG * speed * n.velocity
        n.forces += (0, n.buoyancy, 0)
      IF n is cinecam node 0 AND actor has an engine: engine.stop()     # driver's head under water stalls the engine
      n.under_water = (result)
```

**Notes** — positions are stored relative to a per-actor **origin** that is periodically re-centred ([`UpdatePhysicsOrigin`](Actor.cpp.md)) so single-precision floats keep millimetre precision on large maps. Gravity is applied as a force (mass × g) at the start of the accumulator, so every other force is added on top.

## `CalcBeams(trigger_hooks)` — the spring network

**Contract** — for every enabled beam that is not inter-actor: compute the spring-damper force, apply shock/support/rope/trigger special behaviour, deform plastically when overloaded, break when over strength, and apply equal and opposite forces to the two nodes.

```text
FOR EACH beam b (not disabled, not inter-actor)
  dis   = b.p1.rel - b.p2.rel
  len   = |dis|                          # via fast inverse sqrt
  delta = len - b.L                      # >0 stretched, <0 compressed
  k, d  = b.k, b.d
  v     = dot(b.p1.vel - b.p2.vel, dis) / len     # rate of stretch
  CASE b.bounded OF
    SHOCK1:                              # 'shocks' and wheel beams: hard bump-stop outside bounds
      over = delta - longbound*L   IF delta >  longbound*L
           = -delta - shortbound*L IF delta < -shortbound*L
           = 0 otherwise
      IF over != 0
        (ts, td) = (shock.sbd_spring, shock.sbd_damp) IF b is HYDRO-type ELSE (DEFAULT_SPRING, DEFAULT_DAMP)
        k += (ts - k) * over;  d += (td - d) * over     # blend toward stiff defaults, proportional to overshoot in metres
    TRIGGER: calc_triggers(b, delta, trigger_hooks)
    SHOCK2:  calc_shocks2(b, delta, &k, &d, v)
    SHOCK3:  calc_shocks3(b, delta, &k, &d, v)
    SUPPORTBEAM:
      IF delta > 0                                     # only resists compression
        k = 0; d *= 0.1
        limit = longbound IF longbound > 0 ELSE 4.0
        IF delta > L * limit: break and disable b      # pulled apart too far
    ROPE:
      IF delta < 0: k = 0; d *= 0.1                    # only resists tension
  IF on the frame's last step AND b is a bounded HYDRO beam: record debug k·|delta|, d·|v|, |v|
  stress = -k*delta - d*v
  b.stress = stress
  mag = |stress|
  IF mag > b.minmaxposnegstress                        # cheap pre-check
    IF b is NORMAL AND not SHOCK1 AND k != 0
      deform(b, stress, delta, k)                      # see below; may reduce stress and mag
    IF mag > b.strength
      play break sound (volume ∝ ½·k·delta²)
      IF NOT (p1 is a cab node with < 3 active beams OR p2 is a cab node with < 3 active beams)
        stress = 0; b.broken = b.disabled = true
        IF b.detacher_group > 0                        # master detacher beam
          break every beam whose |detacher_group| == group
          detach every wheel whose detacher group == group
      ELSE
        b.strength = 2 * b.minmaxposnegstress           # protect triangle integrity: make it tougher instead
      IF b's both nodes lie on one buoyant (non drag-only) cab triangle: buoyancy.sink = true
  f = dis * (stress / len)
  b.p1.forces += f;  b.p2.forces -= f
```

`calc_shocks2` / `calc_shocks3` are the progressive shock laws in [`Actor.cpp` — Shock laws](Actor.cpp.md#shock-laws); `calc_triggers` is [`CalcTriggers`](Actor.cpp.md#calctriggersbeam-delta-trigger_hooks). Constants are in [`SimConstants.h`](SimConstants.h.md).

### Plastic deformation

```text
FUNCTION deform(b, stress, delta, k)
  IF stress > b.maxposstress AND delta < 0                    # compressed past yield
    yield = b.maxposstress / k
    L_old = b.L
    b.L  += delta + yield * (1 - b.plastic_coef)              # rest length follows the compression
    b.L   = max(MIN_BEAM_LENGTH, b.L)
    stress -= (stress - b.maxposstress) * 0.5;  mag = stress
    IF b.L < L_old: b.maxposstress *= L_old / b.L; recompute minmaxposnegstress
    # strength is NOT reduced under compression (stability)
  ELSE IF stress < b.maxnegstress AND delta > 0               # stretched past yield
    yield = b.maxnegstress / k
    L_old = b.L
    b.L  += delta + yield * (1 - b.plastic_coef)
    stress -= (stress - b.maxnegstress) * 0.5;  mag = -stress
    IF b.L > L_old: b.maxnegstress *= b.L / L_old; recompute minmaxposnegstress
    b.strength -= (delta + yield*(1-plastic_coef)) * k        # stretching weakens the beam
```

`minmaxposnegstress = min(maxposstress, -maxnegstress, strength)` after each change. Beams are thus **work-hardened**: every deformation raises the yield stress in that direction, so crumple zones progress instead of collapsing at once.

**Notes** — the "don't break if it would leave a cab node with < 3 beams" rule keeps collision triangles attached; vehicles rely on it to not shed their bodywork. Diagnostic logging (break/deform) is controlled by `diag_log_beam_break` / `diag_log_beam_deform`.

## `CalcBeamsInterActor`

**Contract** — the same law for beams whose second node belongs to another actor (hooks, ties, ropes linked across actors), using **absolute** positions, with rope slack and the same deformation/breaking rules but **without** detacher groups or buoyancy sinking. Runs single-threaded between the parallel phases because it writes to two actors.

## `CalcDifferentials`

```text
IF engine AND propelled_wheels > 0
  t = engine.torque / propelled_wheels * (2 IF actor has an 'axles' section ELSE 1)   # legacy compatibility
  add t to every propelled, attached wheel
n_axle_diffs = axle diff count (+1 if transfer case is in 4WD)
detached wheels copy the speed of their partner (per axle diff and per wheel diff) so diffs stay stable
FOR EACH inter-axle differential: combine mean speeds of both axles, total torque of both axles,
    ask the differential for split torques (see Differentials), write half to each wheel of each axle
FOR EACH inter-wheel differential: split the pair's total torque between its two wheels
```

## `CalcWheels(doUpdate, num_steps)`

```text
advance traction-control and anti-lock pulse timers; toggle each pulse state when its timer exceeds the pulse time
wheel_speed = wheel_spin = 0; abs_active = tc_active = false
FOR EACH attached wheel w
  vehicle_speed = dot(node0.velocity, forward direction); cur = |vehicle_speed|
  slip = |w.speed - vehicle_speed| / max(1, cur)
  # traction control
  IF tc enabled AND |w.torque| > 0 AND |w.speed| > cur AND slip > tc_wheelslip_constant
    IF tc pulse on: w.tc_coef = (cur / |w.speed|) ^ tc_ratio
    w.torque *= w.tc_coef ^ min(|w.speed| / 5, 1)
    tc_active = true
  ELSE w.tc_coef = 1
  # brakes
  IF w.braking != NONE
    foot = brake_force * brake_input
    hand = handbrake_force IF parking brake AND w.braking != FOOT_ONLY ELSE 0
    dir  = brake_force * |steering| IF w.speed < 20 AND ((SKID_LEFT AND steering > 0) OR (SKID_RIGHT AND steering < 0)) ELSE 0
    IF any > 0
      fd = foot + dir
      IF abs enabled AND cur > alb_minspeed AND cur > |w.speed| AND fd > 0 AND slip > 0.25
        IF abs pulse on: w.alb_coef = (|w.speed| / cur) ^ alb_ratio
        fd *= w.alb_coef; abs_active = true
      stop = -w.avg_speed * w.radius * w.mass / DT - w.last_retorque     # torque that would stop the wheel in one step
      IF w.speed > 0: w.torque += clamp(stop, -(fd + hand), 0)
      ELSE:           w.torque += clamp(stop, 0, +(fd + hand))
    ELSE w.alb_coef = 1
  # apply torque around the axle, measure wheel speed
  axis = normalise(axis_node_1.rel - axis_node_0.rel)
  per_node = w.torque / w.num_nodes
  expected = w.speed; w.speed = 0
  FOR j, outer IN w.nodes
    inner = axis_node_1 IF j odd ELSE axis_node_0
    r = outer.rel - inner.rel; inv = 1/|r|
    IF w.propulsion == BACKWARD: r = -r
    tangent = cross(axis, r) * inv
    outer.forces += tangent * per_node * inv
    w.speed += dot(outer.velocity - inner.velocity, tangent)
    accumulate contact slip/force of nodes touching ground (for debug display)
  w.speed /= w.num_nodes
  w.net_rp += w.speed / w.radius * DT
  w.avg_speed = w.avg_speed * 0.99 + w.speed * 0.1          # deliberately over-weighted (improves brake estimate)
  IF w.propulsion == FORWARD
    wheel_speed += w.speed / propelled_wheels; wheel_spin += w.speed / w.radius / propelled_wheels
  expected += (w.last_torque / w.radius) / w.mass * DT
  w.last_retorque = w.mass * (w.speed - expected) / DT        # external (ground) torque estimate
  # reaction torque on the suspension (so driving/braking pitches the body)
  rr = arm_node.rel - near_attach_node.rel
  r  = rr projected onto the plane ⟂ axis through near_attach_node
  err = |rr - r|; rlen = |r|
  IF rlen > 0.01 AND 2*err < rlen AND |w.torque| > 0.01
    c = cross(axis, r/rlen) * (0.5 * w.torque / rlen) * (1 - 2*err/rlen)
    arm_node.forces -= c; near_attach_node.forces += c
  w.last_torque = w.torque; w.torque = 0
avg_wheel_speed = avg*0.995 + wheel_speed*0.005
engine.set_wheel_spin(wheel_spin * RAD_PER_SEC_TO_RPM)
on doUpdate: start/stop ABS and TC sounds
odometers += |wheel_speed| * DT
```

## `CalcShocks` — active stabilisers

**Contract** — actors with left/right "active" shocks (`L`/`R` options) level themselves in corners.

```text
IF has active shocks AND stabiliser request != 0
  IF (request == 1 AND ratio < 0.1) OR (request == -1 AND ratio > -0.1)
    ratio += request * DT * STAB_RATE
  right-active shocks: L = refL * (1 + ratio); left-active: L = refL * (1 - ratio)
IF has active shocks AND doUpdate
  sleep -= DT * num_steps
  roll = asin(dot(camera_roll_axis, UP))
  IF |roll| > 0.2: sleep = -1                   # emergency: act immediately
  IF |roll| > 0.01 AND sleep < 0
    request = +1 if roll > 0 and request != -1; -1 if roll < 0 and request != +1; else 0 and sleep = 3 s
  ELSE request = 0
  play/stop the air-valve sound while correcting
```

## `CalcHydros` — steering and control actuators

```text
# steering input -> steering state
IF dir_state != 0 OR dir_command != 0
  IF NOT speed_coupling                           # analog-friendly smoothing
    smooth = clamp(io_analog_smoothing, 0.5, 2); sens = clamp(io_analog_sensitivity, 0.5, 2)
    diff = command - state
    state += (10 / smooth) * DT * exp(-min(|diff|, 1) / sens) * diff
  ELSE
    IF command != 0
      rate = 3.0 IF NOT io_hydro_coupling ELSE max(1.2, 30 / (10 + |wheel_speed| / 2))   # slower steering at speed
      state moves toward command by DT * rate
    state moves toward 0 by DT (self-centring), snapping to 0 within DT
# aileron, rudder, elevator: state moves toward command at 4/s, and toward 0 at 1/s, snapping within DT
FOR EACH hydro h
  c = 0; n = 0
  IF SPEED:     c += dir_state * (12 - wheel_speed) / 12 IF wheel_speed < 12;  n += 1   # speed-sensitive steering
  IF DIR:       c += dir_state; n += 1
  IF AILERON:   c += aileron_state; n += 1     # likewise RUDDER, ELEVATOR
  IF REV_AILERON: c -= aileron_state; n += 1   # likewise REV_RUDDER, REV_ELEVATOR
  c = clamp(c, -1, 1)
  IF h has animator flags: calc_animators(h, &c, &n)      # see Actor.cpp
  IF n > 0
    c = h.inertia.delay(c / n, DT)
    IF NOT SPEED AND no animator flags: dir_wheel_display = c
    factor = 1 - c * h.speed
    IF animator: factor = clamp(factor, 1 - beam.shortbound, 1 + beam.longbound)
    beam.L = h.ref_length * factor
```

## `CalcCommands` — hydraulic commands and rotators

```text
IF actor has command beams
  hydraulics_ready = engine.rpm > 0.95 * idle_rpm (or true without engine)
  crank = engine.crank_factor (1 without engine); crank = 2 for MACHINE actors
  clear auto_move_lock on all command beams
  FOR key IN 1..84
    old = value; value = max(player_input, trigger_input); trigger_input = 0
    state = 1 if value rose above 0.01, -1 if fell below, else unchanged
    IF value >= 0.5: lock auto-move on its beams; auto-centering beams stop auto-moving
  FOR key IN 1..84
    FOR EACH command beam cb of key
      dir = -1 IF cb.is_contraction ELSE +1
      IF cb.force_restricted: crank = min(crank, 1)
      v = value
      IF cb.autocentering AND NOT auto_move_lock
        cur = L / refL
        IF |cur - center| < 0.0001: mode = 0
        ELSE new mode = -1 if cur > center else +1; if it flips sign vs. old mode: snap L = center*refL, mode = 0
      clen = L / refL
      IF (dir > 0 AND clen < boundary) OR (dir < 0 AND clen > boundary)
        one-press-with-centre bookkeeping (pressed_center_mode) and one-press state machine:
          mode 0 → (key down) dir*1 → (key up) dir*2 → (key down) dir*3 → (key up) 0
        v = key.command_inertia.delay(v, DT)
        IF dir * mode > 0: v = 1                     # auto-moving
        IF cb.needs_engine AND (engine stopped OR NOT hydraulics_ready): CONTINUE
        IF v > 0 AND cb.engine_coupling > 0: request power
        command sound start/stop/modulate by rate (per key)
        cf = crank IF cb.engine_coupling > 0 ELSE 1
        L *= 1 ± cb.speed * v * cf * DT / L          # grow or shrink by speed·v·cf m/s
        IF requesting power: active += 1; work += |stress| * |ΔL| * engine_coupling
      ELSE IF one-press AND moving toward this boundary: mode = 0    # reached the end
    FOR EACH rotator r of key (index sign = direction)
      skip if needs engine and not available
      v = key.rotator_inertia.delay(value, DT)
      r.angle ± = r.rate * v * cf * DT
  engine.set_hydro_pump(work); engine.set_prime(any requested)
  player actor, last step: pump sound at 660 * (1 - (work/active)/100) rpm while active
  FOR EACH rotator r                                  # enforce the rotator angle with forces
    axis = normalise(node(axis1) - node(axis2))
    FOR k IN 0, 1
      ref1 = project(axis1 - base_plate[k], plane ⟂ axis); ref2 = project(axis2 - rotating_plate[k], plane ⟂ axis)
      len1, len2 = normalise both
      target = rotate(ref1, angle + π/2, about axis)
      err = asin(dot(target, ref2))
      len1 = 0 IF len1 <= tolerance; len2 = 0 IF len2 <= tolerance   # jitter fix
      base_plate[k].forces   += err*len1*force * cross(ref1, axis)
      rotating[k].forces     -= err*len2*force * cross(ref2, axis)
      base_plate[k+2].forces -= err*len1*force * cross(ref1, axis)   # opposite corners: symmetric
      rotating[k+2].forces   += err*len2*force * cross(ref2, axis)
```

## `CalcTies`

**Contract** — ties that are currently *tying* shorten at `contract_speed` (m/s) until their length fraction reaches `min_length`, or stop early when |stress| exceeds `max_stress`.

## `CalcHooks`

```text
FOR EACH hook h
  h.timer = max(0, h.timer - DT)
  IF h.lock_node AND h.state == PRELOCK
    IF h.beam.L < h.min_length: h.state = LOCKED
    ELSE IF h.beam.L > h.lockspeed AND |h.beam.stress| < h.maxforce: h.beam.L -= h.lockspeed   # reel in
    ELSE IF |stress| < maxforce: h.beam.L = 0.001; h.state = LOCKED
    ELSE IF h.nodisable: h.state = LOCKED
    ELSE queue HOOK_UNLOCK                                       # too much force: let go
```

## `CalcRopes`

**Contract** — for every locked rope, the rope's end node is teleported onto the locked ropable node (position and velocity), the force it accumulated is transferred to the ropable node, and its own force is zeroed — i.e. the rope end is rigidly pinned to the other object.

## `CalcAircraftForces` · `CalcFuseDrag`

**Contract** — airbrakes, aero engines (props/jets), screw props and wings each add their forces (see `air/`, `water/`, `flex/`). Fuselage drag: with a fuselage airfoil defined, `wind = -front.velocity`, reference area `s = |front - back| * width`, angle of attack from the rotation between fuselage axis and wind, airfoil coefficients at that angle, air density from the tropospheric model `ρ = 101325·(1 − 0.0065·alt/288.1)^5.24947 · 1.20896e-5` (≈ 1.225 at sea level), and

`fuse_drag = ((cx·s + width²·0.5) · 0.5 · ρ · |wind| / node_count) · wind`

applied to **every node** (in `CalcNodes`) instead of the default drag.

## `CalcBuoyance`

**Contract** — buoyant cab triangles are expensive, so buoyancy is **sampled every 10 steps** (200 Hz) and interpolated between:

```text
IF step % 10 == 0
  snapshot each buoyant node's position/velocity (current), and a projected copy advanced by velocity * 10·DT
  compute buoyancy forces for every buoyant triangle on both snapshots (visual debug only every 40 steps)
  apply the current forces; remember this step
ELSE
  t = (step - last_sample) / 10
  apply force = current*(1-t) + projected*t for each node
step += 1
```

## `CalcEventBoxes`

**Contract** — asks the collision system for event boxes touching the actor's bounding box. For each: if a node is already recorded inside it, re-test only that node; otherwise scan nodes until one is inside (for `truck_wheels` boxes only tyre nodes count). Newly entered boxes queue the box's legacy script callback and fire `EVENTBOX_ENTER(actor, node, instance, box)`; boxes left fire `EVENTBOX_EXIT`. All callbacks are **queued as messages**, because this runs on physics worker threads.

## Small steps

- `CalcMouse` — while a node is grabbed: `force += grab_force * (target - node.abs)`.
- `CalcTruckEngine` — `engine.update(DT, doUpdate)`.
- `CalcReplay` — tell the replay recorder a step passed.
- `CalcForceFeedback` — for the player's actor: accumulate forces on the current cinecam node, and Σ `speed · refL · stress` over intact steering hydros; reset the accumulators on the frame's last step.
