# source/main/physics/ActorManager.cpp

> Spawn post-processing, the per-frame update and the 2 kHz step loop, sleeping/waking, multiplayer stream handling, truck-file loading, and free forces.

**Needs** — [`ActorManager.h`](ActorManager.h.md) · [`Actor.h`](Actor.h.md) · [`Application.h`](../Application.h.md) · [`ApproxMath.h`](ApproxMath.h.md) · [`water/Buoyance.h`](water/Buoyance.h.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`resources/ContentManager.h`](../resources/ContentManager.h.md) · [`gameplay/ChatSystem.h`](../gameplay/ChatSystem.h.md) · [`collision/Collisions.h`](collision/Collisions.h.md) · [`gui/DashBoardManager.h`](../gui/DashBoardManager.h.md) · [`collision/DynamicCollisions.h`](collision/DynamicCollisions.h.md) · [`gameplay/Engine.h`](../gameplay/Engine.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`system/Console.h`](../system/Console.h.md) · [`gui/panels/GUI_TopMenubar.h`](../gui/panels/GUI_TopMenubar.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`gfx/MovableText.h`](../gfx/MovableText.h.md) · [`network/Network.h`](../network/Network.h.md) · [`collision/PointColDetector.h`](collision/PointColDetector.h.md) · [`gameplay/Replay.h`](../gameplay/Replay.h.md) · [`resources/rig_def_fileformat/RigDef_Validator.h`](../resources/rig_def_fileformat/RigDef_Validator.h.md) · [`resources/rig_def_fileformat/RigDef_Serializer.h`](../resources/rig_def_fileformat/RigDef_Serializer.h.md) · [`ActorSpawner.h`](ActorSpawner.h.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`threadpool/ThreadPool.h`](../threadpool/ThreadPool.h.md) · [`resources/tuneup_fileformat/TuneupFileFormat.h`](../resources/tuneup_fileformat/TuneupFileFormat.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [`gameplay/VehicleAI.h`](../gameplay/VehicleAI.h.md)
**Used by** — callers of [`ActorManager.h`](ActorManager.h.md) (see its Used by)
**Tier floor** — T2

## Purpose

The frame-level orchestration of the simulation. Per-step physics of one actor lives in [`ActorForcesEuler.cpp`](ActorForcesEuler.cpp.md); this file decides *how many* steps run, *which* actors run them, *in what parallel structure*, and what happens around each frame.

## State

See [`ActorManager.h`](ActorManager.h.md).

## `UpdateActors(player)`

**Contract** — called once per rendered frame on the main thread (after input). Converts wall time into a whole number of physics steps, runs the per-frame (non-physics) updates of every actor, then launches the physics steps on the simulation thread. With async physics off it waits for them; with it on they overlap the next frame's rendering and the next call to anything that touches actor state first calls `SyncWithSimThread`.

```text
FUNCTION update_actors(player)
  dt = min(simulation_time, 1/20) * simulation_speed + dt_remainder
  physics_steps = floor(dt / PHYSICS_DT)
  IF physics_steps == 0: RETURN                     # keep remainder accumulating
  dt_remainder = dt - physics_steps * PHYSICS_DT
  dt = physics_steps * PHYSICS_DT
  sync_with_sim_thread()
  update_sleeping_state(player, dt)
  FOR EACH actor
    actor.handle_input_events(dt); actor.handle_script_events(dt)
    IF AI active: ai.update(dt)
    IF actor has engine
      IF actor is a truck: update_truck_features(actor, dt)
      IF actor sleeping: engine.update(dt, 1)      # idle engine keeps running while asleep
      engine.update_audio()
    actor.update_dashboards(dt); actor.update_flare_states(dt)   # always: 'u' flares and blinkers
    IF NOT sleeping: actor.update_visual(dt); IF physics ran AND skidmarks enabled: update skidmarks
    IF multiplayer: remote → actor.calc_network()  (also for hidden remotes); local → actor.send_stream_data()
  IF player
    forward_commands(player)
    player tie/rope toggle flags → queue TIE_TOGGLE / ROPE_TOGGLE (group -1) linking requests; clear flags
    player.force_feedback_step(physics_steps)
    IF player is replaying: advance replay
  sim_task = sim_pool.run(update_physics_simulation)
  total_sim_time += dt
  IF NOT app_async_physics: wait sim_task
```

## `UpdatePhysicsSimulation`

**Contract** — runs on the simulation thread. Executes `physics_steps` steps of all awake actors, using the global thread pool for intra-actor work and collisions.

```text
FOR EACH actor: actor.update_physics_origin()
REPEAT i IN 0 .. physics_steps-1
  jobs = []
  FOR EACH actor
    actor.update_physics = actor.calc_forces_euler_prepare(first_step = (i == 0))
    IF actor.update_physics: jobs += { actor.calc_forces_euler_compute(i == 0, physics_steps) }
  thread_pool.parallelize(jobs)                  # one job per actor
  FOR EACH actor with update_physics: actor.calc_beams_inter_actor()     # serial: touches two actors
  jobs = []
  FOR EACH actor with an inter-actor point detector AND (update_physics OR (mp_pseudo_collisions AND remote))
    jobs += { detector.update_inter_point(); IF actor.collision_relevant: resolve_inter_actor_collisions(PHYSICS_DT, …) }
  thread_pool.parallelize(jobs)
  calc_free_forces()                              # separate serial pass
FOR EACH actor
  ongoing_reset = false
  IF update_physics AND physics_steps > 0
    camera_gforces = 0.5·camera_gforces + 0.5·(gforce_accu / physics_steps); gforce_accu = 0
    calculate_local_gforces(); calculate_average_position()
    avg_velocity = (avg_pos - avg_pos_prev) / (physics_steps · PHYSICS_DT); avg_pos_prev = avg_pos
    top_speed = max(top_speed, |node0.velocity|)
```

**Notes** — the per-step barrier after each parallel phase is load-bearing: inter-actor beams and inter-actor collisions read and write nodes of two actors, so they must not overlap any actor's own step.

## `UpdateInputEvents(dt)`

**Contract** — reads the simulation-pace hotkeys (only when no race is in progress): accelerate multiplies speed by `2^(dt/2)` while held, decelerate by `0.5^(dt/2)`; "reset pace" toggles between 1.0 and the last non-1 speed; starting a race forces speed 1 (remembering the old one). "Toggle physics" pauses/unpauses. Sets `simulation_time = dt`, or when paused 0 — except that while paused with speed > 0, holding "fast forward" or pressing "step forward" yields exactly one physics step (`PHYSICS_DT / speed`).

## `UpdateSleepingState(player, dt)`

**Contract** — actors go to sleep after **10 s** below 0.1 m/s (`|v|² > 0.01` resets the counter); AI-driven actors never sleep; nothing sleeps while forced awake. The player's actor is always woken. Then wakes by contact propagation:

```text
visited = all false
IF player simulated: player.sleep_counter = 0; recursive_activation(player)
FOR EACH simulated actor with sleep_counter == 0: recursive_activation(it)     # snowball

FUNCTION recursive_activation(j)
  IF visited[j] OR actor j not simulated: RETURN
  visited[j] = true
  FOR EACH other actor t not visited
    IF t simulated AND collision boxes of t and j intersect: t.sleep_counter = 0; recursive_activation(t)
    IF t sleeping AND predicted boxes of t and j intersect: wake t; recursive_activation(t)
```

Box tests use the per-collision-group boxes when an actor has them (any pair intersecting), else the whole-actor box.

## `ForwardCommands(source)`

**Contract** — if the source actor has `forwardcommands`: every other actor with `importcommands` within the sum of both minimum camera radii is woken, and (when `sim_realistic_commands` is on, only if it is linked to the source) receives `max(player input, command value)` of each of the source's command keys 1..MAX_COMMANDS, plus tie/rope toggles as linking requests. Independently, every actor locked by one of the source's hooks receives the source's brake, the source's *trailer* parking-brake state (toggled to match), and the source's light mask via import.

## `UpdateTruckFeatures(vehicle, dt)`

**Contract** — skipped while resetting, paused, or AI-driven. With engine contact in automatic mode and not in neutral:
- **Hill hold** — if |pitch| > 2° and the vehicle is rolling back against the selected direction (speed < 0.02 m/s forward gear uphill, or > −0.02 reverse downhill): `ratio = max(0, 1 − (|torque| / avg propelled radius) / (|sin pitch| · mass · g))`, scaled by `sqrt((0.02 − speed)/0.02)` if moving the wrong way, and `brake = sqrt(ratio)`.
- **Creep brake** — on level ground with no brake, no parking brake and zero torque: `brake = max(0, 0.2 − |speed|)/0.2`.

Then cruise control update if on; speed limiter: `throttle = clamp((limit − |wheel_speed|/1.02)·2, 0, current throttle)` when in gear. Sets brake-light bit (`brake > 0.01` and no parking brake) and reverse-light bit (gear < 0).

## `CreateNewActor(request, document)`

**Contract** — allocates the next instance id if the request has none, builds the actor with [`ActorSpawner`](ActorSpawner.cpp.md) (sections, add-on parts, asset packs, then processing), and finishes it. Returns the actor in state LOCAL_SLEEPING (or NETWORKED_OK for remote spawns); appends it to the list.

Post-processing, in order:
1. announce a local stream if connected (`sendStreamSetup`);
2. rotate all nodes about the spawn position by the spawn rotation;
3. **placement** — actors without `fixes`: shift so the bounding-box centre lands on the requested x/z, then `resetPosition(x, z, miny)` where `miny` is the requested y (unless preloaded with terrain → 0) or the spawn box floor; if a spawn box is given and not every node is inside it (0.2 m tolerance), move sideways by 0.6 × (box width + actor width) and place again. Free-position requests and actors with fixes are placed exactly;
4. `recalculateNodeMasses`, store originals (total, dry, load, minimass, per-node);
5. default sound sources unless disabled; node connectivity graph; average position;
6. `min_camera_radius = 1.2 × max distance of any node from the average position`;
7. submesh ground model from name (default if unknown);
8. record per-beam initial strength, default deform, and (k, d);
9. spawn heading; fire "new truck"; count tyre nodes; `first_wheel_node` = first tyre or rim node (network cut-off);
10. initial visuals — full graphics update only when the actor will not be driven immediately (preloaded, from config file, or without cinecams);
11. engine: started if `sim_spawn_running` and not preloaded, else off; tyre pressure initialised;
12. multiplayer buffer sizes: `12 + (first_wheel_node − 1)·6` node bytes, `4·wheels`, `ceil(keys/8)` key bytes; replay recorder when replay is enabled and not multiplayer;
13. cache buoyancy nodes (positions must be final).

## `DeleteActorInternal(actor)`

**Contract** — no-op on null or disposed. Waits for the sim thread; in multiplayer sends STREAM_UNREGISTER for local actors, or forgets the peer's time offset when this was that peer's last actor; unloads scripts associated with the actor; removes free forces that reference it; `dispose()`; removes it from the list and renumbers `vector_index`. The object itself survives while scripts hold it.

## `CleanUpSimulation`

**Contract** — deletes all actors (last first) and resets total time, pause, and speed to defaults.

## Multiplayer

### `HandleActorStreamData(packets)`

**Contract** — processes a frame's worth of received packets.

```text
stable-sort packets by source (descending)
collapse each run of consecutive identical-header STREAM_DATA packets to its last one
FOR EACH packet
  STREAM_REGISTER (type 0 = actor):
     name = sanitised, split "bundle:file"
     unknown user → status -1; empty filename → status -1
     ELSE announce "spawned a new vehicle: <file>" in chat;
          find the file in the mod cache by filename only (bundle ignored on purpose)
          missing → console "Mod not installed: <file>", remember as mismatch, status -1
          found  → request_spawn_remote_actor, status 1
     reply STREAM_REGISTER_RESULT with the (modified) register struct
  STREAM_REGISTER_RESULT: store the peer's status on our matching local actor, log
  STREAM_UNREGISTER: delete that remote actor (if OK/hidden), drop mismatch records
  USER_LEAVE: drop all that peer's mismatch records, delete all its remote actors
  STREAM_DATA: forward to the matching NETWORKED_OK actor's push_network
```

### `RequestSpawnRemoteActor(reg, entry, user, peeropts)`

**Contract** — first stream from a peer sets its time offset to `reg.time − net_now − 100` (100 ms of interpolation delay). Queues a spawn request with origin NETWORK, skin (by name, if the field is a terminated string), section config, username, colour, peer options, and (source, stream).

### Other

- `RetryFailedStreamRegistrations(event)` — on "mod cache entry added", every remembered mismatched registration whose filename equals the new entry is removed from the mismatch sets and spawned.
- `GetNetTimeOffset` / `UpdateNetTimeOffset(source, delta)` — delta is added only if the peer already has an offset.
- `CheckNetworkStreamsOk(source)` — MISMATCHES if any unresolved; ALL_OK if we have a live remote actor from it; else IDLE. `CheckNetRemoteStreamsOk(source)` — over *local* actors: MISMATCHES if that peer reported −1/−2 for any, ALL_OK if any reported 1, else IDLE.

## Lookup and cycling

- `FindActorInsideBox(collisions, instance, box)` — the unique actor whose node 0 is inside the named event box; none if zero or two or more. `RepairActor` resets that actor on the spot (with repair sound).
- `FetchNext/PreviousVehicleOnList(player, previous)` — pivot = player's index, else previous player's index + 1; search forward (backward), wrapping, finally the pivot itself; excludes terrain-preloaded actors and, unless `mp_cyclethru_net_actors`, remote ones.
- `FetchRescueVehicle` — first actor with the rescuer flag.

## `FetchActorDef(request)`

**Contract** — returns the parsed document for the request's cache entry, parsing it once and caching it on the entry. Loads the entry's resources, opens the file, runs the [parser](../resources/rig_def_fileformat/RigDef_Parser.cpp.md) (prepare, stream, finalize) and the [validator](../resources/rig_def_fileformat/RigDef_Validator.cpp.md) — skipping beam checks for terrain-spawned `.load`/`.fixed` files, which legitimately have no beams — and stores the SHA-1 of the file text as the document hash. Any failure posts "Failed to load '<file>' (type: 'actor'), message: …" to the console and returns nothing.

`ExportActorDef(document, file, group)` serialises with the [serializer](../resources/rig_def_fileformat/RigDef_Serializer.cpp.md) into a new (overwritten) resource; failure goes to the console.

## Free forces

Script-created forces applied once per physics step to one node, after all actor steps:

| Type | Force on base node |
|---|---|
| CONSTANT | magnitude · fixed direction |
| TOWARDS_COORDS | magnitude · unit(target point − node) |
| TOWARDS_NODE | magnitude · unit(target node − node) |
| HALFBEAM_GENERIC / HALFBEAM_ROPE | a one-sided beam to the target node, with the same spring/damper, plastic deformation and breaking law as an inter-actor beam; a rope half-beam has k = 0 and d × 0.1 when slack. Only the base node receives force. Breaking turns it into DUMMY (inert) with the break sound |

Add rejects a duplicate id; modify/remove reject unknown ids; all validate base/target actor (exists, not disposed) and node numbers, printing errors to the console. A half-beam's rest length is the node distance at add/modify time, and deform is its initial yield stress both ways. Each change fires "free forces activity" (added/modified/removed/deformed/broken).
