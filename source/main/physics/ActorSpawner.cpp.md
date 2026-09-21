# source/main/physics/ActorSpawner.cpp

> How each truck-file element becomes nodes, beams, shocks, wheels, differentials, hooks, lights, sounds and visuals.

**Needs** — [`ActorSpawner.h`](ActorSpawner.h.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.h`](../resources/addonpart_fileformat/AddonPartFileFormat.h.md) · [`AppContext.h`](../AppContext.h.md) · [`Application.h`](../Application.h.md) · [`air/AirBrake.h`](air/AirBrake.h.md) · [`air/Airfoil.h`](air/Airfoil.h.md) · [`ApproxMath.h`](ApproxMath.h.md) · [`gameplay/AutoPilot.h`](../gameplay/AutoPilot.h.md) · [`Actor.h`](Actor.h.md) · [`ActorManager.h`](ActorManager.h.md) · [`utils/BitFlags.h`](../utils/BitFlags.h.md) · [`water/Buoyance.h`](water/Buoyance.h.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`gfx/camera/CameraManager.h`](../gfx/camera/CameraManager.h.md) · [`CmdKeyInertia.h`](CmdKeyInertia.h.md) · [`collision/Collisions.h`](collision/Collisions.h.md) · [`gui/DashBoardManager.h`](../gui/DashBoardManager.h.md) · [`Differentials.h`](Differentials.h.md) · [`gameplay/Engine.h`](../gameplay/Engine.h.md) · [`flex/FlexAirfoil.h`](flex/FlexAirfoil.h.md) · [`flex/FlexBody.h`](flex/FlexBody.h.md) · [`flex/FlexMesh.h`](flex/FlexMesh.h.md) · [`flex/FlexMeshWheel.h`](flex/FlexMeshWheel.h.md) · [`flex/FlexObj.h`](flex/FlexObj.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`gfx/GfxActor.h`](../gfx/GfxActor.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`system/Console.h`](../system/Console.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`utils/MeshObject.h`](../utils/MeshObject.h.md) · [`collision/PointColDetector.h`](collision/PointColDetector.h.md) · [`water/ScrewProp.h`](water/ScrewProp.h.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md) · [`gfx/Skidmark.h`](../gfx/Skidmark.h.md) · [`resources/skin_fileformat/SkinFileFormat.h`](../resources/skin_fileformat/SkinFileFormat.h.md) · [`SlideNode.h`](SlideNode.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`gameplay/TorqueCurve.h`](../gameplay/TorqueCurve.h.md) · [`resources/tuneup_fileformat/TuneupFileFormat.h`](../resources/tuneup_fileformat/TuneupFileFormat.h.md) · [`air/TurboJet.h`](air/TurboJet.h.md) · [`air/TurboProp.h`](air/TurboProp.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [`gameplay/VehicleAI.h`](../gameplay/VehicleAI.h.md)
**Used by** — callers of [`ActorSpawner.h`](ActorSpawner.h.md) (see its Used by)
**Tier floor** — T2

## Purpose

The semantic layer of the truck format: the [parser](../resources/rig_def_fileformat/RigDef_Parser.cpp.md) produces records; this file decides what each record *means* physically. Constants referenced here (`BEAM_DEFORM`, `HOOK_*`, `DEFAULT_*`, `MAX_*`) are listed in [`SimConstants.h`](SimConstants.h.md). Material and entity setup is graphics plumbing and is summarised at the end.

## State

See [`ActorSpawner.h`](ActorSpawner.h.md).

## Allocation

### `CalcMemoryRequirements` / `InitializeRig`

**Contract** — arrays are allocated **once, exactly sized**, before any element is processed; nothing grows afterwards (the simulation holds raw references into them). Counts summed over all selected modules:

| Element | Nodes | Beams | Shocks |
|---|---|---|---|
| node (+1 beam if hook flag `h`) | 1 | 0/1 | |
| beams, ties, ropes, hydros, commands2, animators | | 1 each | |
| triggers | | 1 | 1 |
| shocks, shocks2, shocks3 | | 1 | 1 |
| cinecam | 1 | 8 | |
| wheels, meshwheels, meshwheels2 | 2·rays | 8·rays (9 with rigidity node) | |
| wheels2 | 4·rays | 24·rays (25) | |
| flexbodywheels | 4·rays | 20·rays (21) | |

Rotators, wings, airbrakes and fixes are counted likewise. `InitializeRig` also sets defaults: brake force 30 000 N, handbrake 2× brake, speedo max 140 km/h, anti-lock and traction control off with ratio 1 and pulse 2000 Hz, camera slots invalid, state LOCAL_SLEEPING, origin zero; creates the dashboard, the AI controller, and the inter- and intra-actor point collision detectors (unless disabled by `sim_no_collisions` / `sim_no_self_collisions`); loads the flexbody cache.

## Nodes

### `ProcessNode`

```text
index = register(id)            # named: must be unique (duplicate → error, node skipped)
                                # numbered: a number below the current count → warning "previous definition will be overriden",
                                #           but a NEW node is appended anyway; numbers are not positions
spawn_offset = tuneup-tweaked position; absolute = spawn_position + spawn_offset
friction, volume, surface ← node defaults
minimass ← per-node 'set_default_minimass' if active, else global minimass
IF node defaults load_weight >= 0: mass = that, loaded, overridden
ELSE mass = NODE_LOADWEIGHT_DEFAULT, not loaded
lockgroup ← NOLOCK if the file says 'lockgroup_default_nolock', else DEFAULT
options = node options ∪ node-default options
'l': loaded; with explicit weight → overridden mass, else counts toward masscount
'h': create a hook — a disabled rope-type beam to node 0 (node 1 if this is node 0), strength ×100, damping ×0.1,
     L = refL = HOOK_RANGE_DEFAULT, invisible; hook record with defaults (group -1, range, speed, force, lock timer)
buoyancy = 10000 if 'b' else dry_mass / 15
'c' → no ground contact; 'm' → not mouse-grabbable; 'x'/'y' → exhaust point/direction node
track fuselage y/z extents
```

**Notes** — the exhaust node fields are overwritten for *every* node (to 0 when the flag is absent), so only a flagged node that is also the last node survives; the legacy exhaust therefore rarely appears. Recorded as observed behaviour. Buoyancy uses `dry_mass` as known *at that point*, so `globals` must precede `nodes` (it does, by the flow order).

### Node references

Named references resolve through `named_nodes`; numbered references must be `< node count` unless they came through an add-on part import (those pass unchecked). Failures log an error and yield "invalid"; "OrThrow" variants abort the current element. In node *ranges*, a numeric end beyond the count is accepted with an error "for backwards compatibility"; start > end is swapped.

### `ProcessCinecam`

A new node at spawn position + offset, no ground contact, default friction, buoyancy by defaults, global minimass (its configured `node_mass` is applied later by mass recalculation), connected to its 8 listed nodes with beams of the cinecam's spring and damping.

### Others

`fixes` → node immovable. `contacters` → node is a contacter. `lockgroups` → set node lockgroup. `collisionboxes` → assign nodes to a new collision-box id (re-assignment logged). `set_collision_range` → collision range (negative → default 0.02 m).

## Beams

### `AddBeam` (common initialisation)

Every beam starts enabled, with the element's detacher group, strength = `breaking_threshold` (unscaled), plastic coefficient, and the deformation threshold:

```text
FUNCTION deformation_threshold(defaults)
  deform = BEAM_DEFORM; creak = BEAM_CREAK_DEFAULT
  IF defaults are user-defined ('set_beam_defaults' seen)
    deform = defaults.deformation_threshold
    IF NOT enable_advanced_deformation AND deform < BEAM_DEFORM: deform = BEAM_DEFORM
    IF plastic coefficient user-defined AND >= BEAM_PLASTIC_COEF_DEFAULT: creak = 0
  deform = max(deform, creak)
  RETURN deform × defaults.scale.deformation_threshold
# stored as maxposstress = +t, maxnegstress = -t, minmaxposnegstress = t
```

### Per element

| Element | Type | k, d | Strength | Length & bounds |
|---|---|---|---|---|
| beams | normal | scaled defaults | scaled | L = refL = node distance; `r` → rope; `s` → support beam with `longbound` = extension break limit; `i` → invisible |
| shocks | hydro, SHOCK1 | spring, damping | 4 × breaking | L, refL × precompression; bounds as given, or divided by length for `m` (metric); `L`/`R` → active left/right shock |
| shocks2 | hydro, SHOCK2 | in-spring/damp | 4 × breaking | as shocks; `M` (absolute metric) → short = (len − short)/len, long = (long − len)/len, clamped with warnings; `s` → soft-bump; shock stores in/out spring, damp, progression factors |
| shocks3 | hydro, SHOCK3 | in-spring/damp | 4 × breaking | as shocks2; stores in/out split velocities and slow/fast damping |
| commands2 | hydro (`r` → rope) | scaled defaults | scaled | node distance; one command-beam entry on the contract key and one on the extend key (speed, boundary length, force-restricted `f`, auto-centre `c`, needs engine, one-press `p`/`o`, sound, engine coupling, centre = midpoint of the two limits) |
| hydros | hydro | scaled defaults | scaled | node distance; input flags from options (`s` speed-disable, `a` aileron, `r` rudder, `e` elevator, `u` a+e, `v` −a+e, `x` a+r, `y` −a+r, `g` e+r, `h` −e+r, `n` steering) |
| animators | hydro | scaled defaults | scaled | short/long bound default 0.99999 / 10⁶ unless given; source flags → animator flags with parameter (shifter 1–4, altimeter 1–3, aero engine index) |
| triggers | hydro, TRIGGER | 0, 0 | breaking | node distance; see below |
| ropes | hydro, ROPE | scaled | scaled | node distance; visible "tracks/beam"; rope record UNLOCKED, group 0 |
| ties | hydro, ROPE, **disabled** | scaled | scaled | from root node to node 0 (or 1), L = refL = max reach; tie record with contract speed, max stress, min length, no-self-lock |

Commands, rotators and ties mark the actor as having command beams (adds the pump sound). Each distinct (contract key, extend key) pair not described as `"hide"` enters the unique command-pair list; the last non-empty description wins.

### Triggers (`ProcessTrigger`)

Flag mapping: `i` invisible, `x` start disabled, `B` blocker, `b` key blocker (keys start blocked), `s` command switch, `c` command-style bounds (`short = |short − 1|`, `long = long − 1`), `A` inverted blocker, `h`/`H` unlock/lock hook group, `t` continuous, `E` engine trigger. Validation: a plain trigger needs `1 ≤ short action ≤ MAX_COMMANDS`; blockers need non-negative actions; an engine trigger cannot also be a blocker, hook toggle or switch. A plain trigger with long action −1 (and not a hook toggle) is a **command-key blocker**. Boundary timer defaults to 1 s. The shock's `sbd_spring/sbd_damp` come from the unscaled beam defaults. The initial blocked state of both command keys is written into the command-key table (negative indices are valid there). `FinalizeRig` later clamps every blocker's counts so they never reach past the last beam.

### Hooks, ropables, rails, slide nodes

- `hooks` — refine the hook created by the node's `h` flag (error if the node has none): range, speed = factor × default, max force, group, lockgroup, timer preset, min length, self-lock, no-disable; auto-lock sets group −2 if it was −1; `norope` makes the hook beam non-rope; `visible` off removes its visuals.
- `ropables` — record node, group, multilock.
- `railgroups` / slide-node rails — a rail is the chain of existing beams joining consecutive nodes of the list (error and no rail if any pair has no beam); segments are doubly linked, closed into a loop when first node = last node.
- `slidenodes` — spring rate, break force, tolerance, attachment rate and distance when given; constraint flags choose self/foreign attachment; rail from rail-group id or inline ranges (error if neither).

### Rotators

`rotators` / `rotators2` — axis nodes, 4 base-plate and 4 rotating-plate nodes, rate, force and tolerance (defaults for v1), engine coupling. Left key receives `−(n+1)`, right key `+(n+1)` (1-based signed rotator id). Validation warns when either plate is off-centre on the axis (opposite corners' projected distances differ by > 0.1 %) or the plates are misaligned (projected corner dot products differ by > 0.1 %).

### Inertia

A per-command/hydro/rotator inertia is used when both its start and stop delay factors are non-zero, else the `set_inertia_defaults` in force when either default factor is positive, else none. Function names `""`, `"/"`, `"-"` mean "none".

## Wheels

All wheel types first order their axis nodes so that node 1 has the smaller z. A wheel records braking, propulsion, radius, the two axis nodes, the arm node and the *near attach* node (the axis node closer to the arm node). Propelled wheels are appended to the propelled-wheel list, which later drives automatic differentials.

### `wheels`, `meshwheels`, `meshwheels2` (`BuildWheelObjectAndNodes` + `BuildWheelBeams`)

```text
axis = unit(axis2 - axis1); ray = any unit perpendicular to axis × radius
step = rotation by -360/(2·rays) degrees about axis
FOR i IN 0..rays-1
  outer node at axis1 + ray; ray = step(ray)
  inner node at axis2 + ray; ray = step(ray)          # tread nodes alternate sides in a zig-zag
  each: mass = wheel_mass / (2·rays), contacter, tyre node, friction/volume/surface from defaults
  spawn offsets follow the axis nodes' offsets
width = axis length (the file's width is ignored)
FOR each ray i (o = outer, n = inner, o'/n' = next ray)
  axis1–o, axis2–n   : tyre spring/damp, bounded SHOCK1 with shortbound 0.66 and longbound = max_extension
  axis2–o, axis1–n   : tyre spring/damp
  o–n, o–o', n–n', n–o' : rim spring/damp
  rigidity node → (o or n, the side nearest the rigidity node) : virtual beam
```

`wheels` and `meshwheels` use the wheel's spring/damping for both tyre and rim; `meshwheels2` uses the beam defaults for the rim and sets `max_extension` 0.15.

### `wheels2`

Rim ring and tyre ring are generated separately:

```text
rim nodes:  2·rays, starting at (0, rim_radius, 0) from each axis node, step -360/rays; mass = mass/(4·rays); rim nodes; global minimass
tyre nodes: start rotated by -180/rays (half a step); outer mass = 0.67·mass/(2·rays), inner 0.33·mass/(2·rays);
            friction = width × WHEEL_FRICTION_COEF; contacter; tyre node
per ray: rim beams  axis1–o (short 0.66), axis2–n (short 0.66), axis2–o, axis1–n,
                    axis1–o again, o–n, o–o', n–n', o–n', n–o'  (+ rigidity virtual beam)
         tyre beams band (4), sidewalls (4), reinforcement (4), backpressure axis1–to, axis2–ti  → registered with tyre pressure
```

**Notes** — `wheels2` measures width on the already-normalised axis vector, so its width is always 1; and it adds the axis1–outer rim beam twice. Both are original behaviour that affects friction and stiffness; keep them for fidelity.

### `flexbodywheels`

Like `wheels2` but the rim starts perpendicular to the axis and alternates sides with step −360/(2·rays); all nodes get `mass/(4·rays)`; rim beams: 4 spokes + 4 ring beams per ray; tyre beams: 6 rim-to-tread at half tyre spring, 4 tread beams from beam defaults, optional virtual rigidity beam; plus 2 **support beams** per ray from axis to tread, SHOCK1 with `shortbound = 1 − 0.95·rim_radius/tyre_radius`, longbound 0 — they stiffen when the tread would collapse into the rim.

### Differentials

- `axles` — find the wheels whose axis pairs match (warning if not); types from options (`l` locked, `o` open, `s` split, `v` viscous) or default Open+Locked.
- `interaxles` — axle indices must differ and exist, and not coincide with the transfer case pair; default Locked.
- `transfercase` — validates; resets all propulsion, then propels the two wheels of the primary axle (and of the secondary one if 2WD is not offered, starting in 4WD).
- `wheeldetachers` — (wheel id, detacher group) pairs; invalid wheel id → error.

## Drivetrain and assists

- `engine` — creates the engine (shift rpm, torque, gear ratios), type TRUCK, gearbox mode from config. `engoption`, `engturbo`, `torquecurve` require an engine (else warning); torque curve is a named model or custom samples.
- `brakes` — brake force; handbrake 2× unless given.
- `tractioncontrol` / `antilockbrakes` — regulating force clamped to 1..20; pulse rate outside (1, 2000) → 2000 Hz, stored as period; ABS min speed km/h → m/s, at least 0.5; mode/no-dashboard/no-toggle flags.
- `cruisecontrol` — lower speed limit (non-positive accepted with warning), autobrake. `speedlimiter` — enable with max speed.

## Aero and water

- `turbojets` — creates the jet (type AIRPLANE, autopilot for local actors) with nozzle and optional afterburner visuals.
- `turboprops2` / `pistonprops` — a propeller engine (turboprop pitch −10, piston pitch from file); blade props on the reference node are scaled by `distance(ref, blade1)/2.25` and linked to the engine.
- `screwprops` — prop/back/top nodes and power; type BOAT (limit 8).
- `wings` — each wing is a flex airfoil over 8 nodes with texture coordinates, control-surface letter, chord point, deflection limits, airfoil and efficacy. Consecutive wings form one **wing** as long as each new segment's second node equals the previous segment's first-left-down node; at a discontinuity the finished wing gets **induced drag** (span = distance from start segment's right tip to previous segment's left tip, total area) and, the first time on a vehicle without engine, green/red navigation and white strobe lights at the wingtips. Area of a segment = half the sum of the two triangle cross-product magnitudes. Errors: previous wing has no airfoil, empty airfoil name.
- `fusedrag` — fuselage airfoil at the front node; autocalc width = (z extent × y extent) × area coefficient. The "back" node is set to the *front* node too (kept as original, marked "probably a bug").
- `airbrakes` — airbrake panel from four nodes; its visual parts are moved into the graphics actor.
- **Wash** (in finalize): a wing is washed by a propeller if its centre is 0–15 m behind the prop in x and within the prop radius in y; wash ratio = overlap of the wing's z span with the prop's ±radius, divided by the wing span.

## Cabs (`submesh`)

Texcoords and cab triangles append to the old-style cab lists (limits 3000 each). Cab options: `c`/`p`/`u` → collision cab; `b` buoyant, `r` drag-only, `s` no-drag buoyant; `D`/`F`/`S` → both collision and buoyant. Any buoyant cab creates the buoyancy module. `backmesh` duplicates the submesh twice: a transparent copy, then an opaque copy with each triangle's first two vertices swapped (reversed winding).

## Lights and sounds

- **Flares** — skipped when flares are disabled. Blink delay −2 → 0.5 s for blinkers, none otherwise; size −2 → 1 for headlights else 0.5. User flares: control number 12 = legacy parking-brake indicator (becomes a dashboard flare), else clamped into 1..10 and stored 0-based. Dashboard flares need a valid dashboard link (else placeholder). A headlight with a custom material is re-typed as **tail light** (pre-2022 convention). Light sources are created depending on the global flares mode (headlight/high/fog spotlights for the current vehicle; tail/brake/reverse/blinker/user/side lights only in "all vehicles, all lights"). `flares3` adds incandescence inertia with a private material.
- `flaregroups_no_import` — maps flare type to the light-mask bit that linked actors will not import (tail → headlight bit; user n → CUSTOMn).
- **Sound sources** — up to 128 per actor; `soundsources2` with a mode.
- **`SetupDefaultSoundSources`** (called after spawn, unless disabled): engine type `t` → diesel, force, brakes, park brakes, reverse beep; `c` → car; turbo → big/small/mid by turbo inertia (≥3 / ≤0.5) + blow-off + wastegate; air brakes → air purge; starter; turn signal; trucks → horn (police siren if a lightbar exists) and shift; command beams → pump; ABS/TC available → their sounds; wheeled trucks/planes → tyre screech; always break and creak; boats → large marine engine above 50 t else small, started immediately; planes → GPWS callouts, AoA warning, 13 radio chatter clips, and per aero engine (≤8) start/low/high(/afterburner) sets by engine kind; one extend/retract linked sound pair per command beam.

## `FinalizeRig`

```text
torque curve: space samples evenly; if not ascending → error, fall back to default model
gearbox mode ← config
clamp trigger-blocker counts to beams remaining after the blocker
lowest node y → later spawn height helper
main camera nodes ← camera 0 (or node 0)
wheel mass = sum of its tread node masses; average propelled wheel radius
IF no 'axles': make a VISCOUS wheel differential for each consecutive pair of propelled wheels (1–2, 3–4, …)
IF no 'interaxles': make an axle differential between consecutive wheel differentials,
   LOCKED if the file had 'axles' else VISCOUS, skipping the transfer case pair
IF transfer case has a secondary axle: add a LOCKED axle differential for it
IF main camera direction node is 0/invalid: use the node farthest from the camera position and
   store the heading correction quaternion
spawn height = camera node y − lowest node y (0 if no camera)
per camera: if (dir × roll).y > 0 mark roll inverted and warn "camera definition is probably invalid…"
wings: autopilot inertial references; induced drag for the last wing; wash
mark cab-triangle nodes of collision cabs; count contacters; mark contactable nodes:
   not no-contact AND (cab node OR rim node OR the actor has no collision cabs)
save flexbody cache; sort flexbodies
```

## Graphics plumbing

- **Per-actor materials** (`SetupNewEntity`, `FindOrCreateCustomizedMaterial`) — every entity's materials are replaced with actor-private substitutes, one per original name, resolved in this order: simple colour materials (diagnostic mode, colour by component kind); `mirror` (special prop — one clone per mirror); video-camera materials; material-flare bindings; skin *material* replacements; managed materials; otherwise a clone of the original; then skin *texture* replacements and dashboard render-target placeholders (`dashtexture`, `RTTTexture#`) are patched into the substitute.
- **Managed materials** — built from built-in templates `managed/{flexmesh,mesh}_{standard,transparent}/…` chosen by damage/specular maps present and the alternative-materials option; missing diffuse map → skipped; missing optional maps → ignored; placeholders with the original name are registered in every module's group so meshes load; tuneup-removed → translucent red.
- **Props** — reference/x/y nodes, tuned offset/rotation (Z·Y·X Euler), camera visibility mode; special props: mirrors, dashboards (steering wheel mesh at default offset (±0.67, −0.61, 0.24)), driver seat (first only; `seat` forces a see-through material), beacon/redbeacon/lightbar (spot/point lights with rotating flares; lightbar marks the actor as police), aero spinner/blade. Animations map sources and modes to flags; auto-animate without limits uses ±180° / ±10 m; an event source registers a key state (with event-lock) that is sent over the network.
- **Flexbodies** — resolve `forset` and `forvert` nodes; failures create *placeholders* so flexbody ids stay stable (tuning refers to them by index). Tuneup can remove or re-offset/rotate/re-mesh them.
- **Beam visuals** — one rod mesh per visible beam; hydros use chrome; diameter from beam defaults.
- **Video cameras**, mirror cameras, cab mesh (with `-trans`, `-back`, `-noem` material variants), exhausts and particles (only in particle mode 1), skidmarks for every wheel (always created), dashboards (from `guisettings`, else the default truck/boat layouts plus the legacy "renderdash"), help material.

Scene object names are `"<object>#<n> (<truck file> [Instance ID <id>])"` because the renderer requires globally unique names.
