# source/main/gameplay/Replay.h

> A ring buffer of recent actor states (node positions/velocities, beam broken/disabled) for rewinding.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`GameContext.cpp`](../GameContext.cpp.md) · [`Replay.cpp`](Replay.cpp.md) · [`gfx/camera/CameraManager.cpp`](../gfx/camera/CameraManager.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/ActorForcesEuler.cpp`](../physics/ActorForcesEuler.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md)
**Tier floor** — T2


## Purpose

Created per local actor when `sim_replay_enabled` is on (never in multiplayer). Implementation: [`Replay.cpp`](Replay.cpp.md).

## State

```text
RECORD node_simple = { position, velocity : Vec3 }
RECORD beam_simple = { broken, disabled : bit }
RECORD Replay
  actor; frames : int (sim_replay_length); precision : s = 1/sim_replay_stepping (0 = every step)
  nodes[frames × node_count], beams[frames × beam_count], times[frames] (µs)   # allocated on first write
  write_index, first_run (buffer not yet wrapped), out_of_memory
  timer (accumulates physics time), position (0 = newest, negative = back in time), previous position, current frame time
```

## API

`onPhysicsStep`, `replayStepActor`, `UpdateInputEvents`, buffer access (`getWriteBuffer(type)`, `writeDone`, `getReadBuffer(offset, type, out time)`), getters (precision, position seconds, frame count, current frame, valid).
