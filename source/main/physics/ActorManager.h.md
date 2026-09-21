# source/main/physics/ActorManager.h

> The registry of all actors and the owner of the simulation clock: spawning, deletion, sleeping, stepping, network streams, free forces, savegames.

**Needs** — [`Application.h`](../Application.h.md) · [`CmdKeyInertia.h`](CmdKeyInertia.h.md) · [`network/Network.h`](../network/Network.h.md) · [`resources/rig_def_fileformat/RigDef_Prerequisites.h`](../resources/rig_def_fileformat/RigDef_Prerequisites.h.md) · [`gameplay/ScriptEvents.h`](../gameplay/ScriptEvents.h.md) · [`SimData.h`](SimData.h.md) · [`threadpool/ThreadPool.h`](../threadpool/ThreadPool.h.md)
**Used by** — [`GameContext.h`](../GameContext.h.md) · [`gameplay/Character.cpp`](../gameplay/Character.cpp.md) · [`gameplay/Engine.cpp`](../gameplay/Engine.cpp.md) · [`gameplay/Replay.cpp`](../gameplay/Replay.cpp.md) · [`gfx/GfxScene.cpp`](../gfx/GfxScene.cpp.md) · [`gfx/camera/CameraManager.cpp`](../gfx/camera/CameraManager.cpp.md) · [`gui/GUIManager.cpp`](../gui/GUIManager.cpp.md) · [`gui/OverlayWrapper.cpp`](../gui/OverlayWrapper.cpp.md) · [`gui/panels/GUI_MainSelector.cpp`](../gui/panels/GUI_MainSelector.cpp.md) · [`gui/panels/GUI_MultiplayerClientList.cpp`](../gui/panels/GUI_MultiplayerClientList.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`network/OutGauge.cpp`](../network/OutGauge.cpp.md) · [`Actor.cpp`](Actor.cpp.md) · [`ActorForcesEuler.cpp`](ActorForcesEuler.cpp.md) · [`ActorManager.cpp`](ActorManager.cpp.md) · [`ActorSpawner.cpp`](ActorSpawner.cpp.md) · [`physics/collision/Collisions.cpp`](collision/Collisions.cpp.md) · [`physics/collision/PointColDetector.cpp`](collision/PointColDetector.cpp.md) · [`physics/water/Buoyance.cpp`](water/Buoyance.cpp.md) · [`physics/water/ScrewProp.cpp`](water/ScrewProp.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`scripting/ScriptEngine.cpp`](../scripting/ScriptEngine.cpp.md) · [`system/ConsoleCmd.cpp`](../system/ConsoleCmd.cpp.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md)
**Tier floor** — T2

## Purpose

One instance lives in the game context. It owns the ordered list of actors (the order defines each actor's `vector_index` and the hotkey cycling order), the global table of inter-actor links, script-created free forces, and the one-thread pool on which physics runs. Implementation in [`ActorManager.cpp`](ActorManager.cpp.md); savegames in [`Savegame.cpp`](Savegame.cpp.md).

## State

```text
RECORD ActorManager
  actors               : list<Actor>          # index == actor.vector_index (renumbered after deletion)
  next_instance_id     : int = 1              # unique per session, never reused
  inter_actor_links    : map<beam, (Actor, Actor)>   # every hook/tie/rope beam currently joining two actors
  free_forces          : list<FreeForce>; next_free_force_id = 0
  forced_awake         : bool                 # disables sleep counting
  physics_steps        : int                  # steps to run in the current frame
  dt_remainder         : seconds              # carried rounding error (< PHYSICS_DT)
  simulation_speed     : float >= 0 = 1       # slow/fast motion factor
  last_simulation_speed: float = 0.1          # restored by "reset pace" toggle
  simulation_time      : seconds              # time to advance this frame (0 when paused)
  simulation_paused    : bool; total_sim_time : seconds
  stream_mismatches    : map<source, set<stream>>          # remote streams we couldn't spawn
  mismatched_regs      : list<ActorStreamRegister>         # kept to retry after a mod download
  stream_time_offsets  : map<source, ms>                   # per-peer clock offset for interpolation
  net_timer            : ms clock
  sim_pool             : ThreadPool(1 worker); sim_task : pending task handle
  inertia_config       : CmdKeyInertiaConfig               # shared command-key inertia curves
```

## API

- **Lifetime** — `CreateNewActor(request, document)`, `DeleteActorInternal(actor)` (only via the delete-actor message).
- **Lookup** — by instance id, by (source, stream), the single actor in a named box, next/previous in list, rescue vehicle, nearest to a point, local (non-remote) actors, "directly linked?".
- **Free forces** — add, modify, remove, find, list, next id.
- **Stepping** — `UpdateActors(player)`, `UpdatePhysicsSimulation`, `SyncWithSimThread`, sleep control, speed/pause, total time, `UpdateInputEvents(dt)`, `CleanUpSimulation`.
- **Truck files** — `FetchActorDef(request)`, `ExportActorDef(document, filename, group)`.
- **Networking** — stream data dispatch, net time and per-peer offsets, mismatch bookkeeping, retrying registrations when a mod appears, spawn remote actor, stream health for the UI.
- **Savegames** — `LoadScene`, `SaveScene`, `RestoreSavedState` (see [`Savegame.cpp`](Savegame.cpp.md)).
