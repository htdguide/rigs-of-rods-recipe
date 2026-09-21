# source/main/gameplay/Replay.cpp

> Recording actor snapshots at a fixed cadence and scrubbing through them.

**Needs** — [`Replay.h`](Replay.h.md) · [`Application.h`](../Application.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`utils/Utils.h`](../utils/Utils.h.md)
**Used by** — callers of [`Replay.h`](Replay.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

"Replay mode" freezes the actor (state LOCAL_REPLAY, not simulated) and poses it from the ring buffer.

## State

See [`Replay.h`](Replay.h.md).

## Recording — `onPhysicsStep`

Every physics step add `PHYSICS_DT` to the timer; when it reaches the precision, write one frame (all node positions and velocities, all beam flags, the wall-clock time) and reset the timer. Memory is claimed lazily on the first write; any allocation failure disables recording for good. The log reports the buffer size up front.

## Reading — `getReadBuffer(offset, type)`

Offset is clamped to `−frames + 1 … −1` (0 or positive means "the newest"). Index = write_index + offset, wrapping backward unless the buffer has never wrapped (then clamp to the oldest frame 0).

## `replayStepActor` (per frame, player's actor in replay)

When the position changed: copy positions (and relative positions), velocities, zero forces; refresh slide nodes, bounding boxes, average position; copy beam flags.

## `UpdateInputEvents`

Toggle replay mode; in replay: forward/backward ±1 and fast ±10 frames (bounded to −frames..0); with left Alt held, mouse X movement scrubs (×0.05, or ×1.5 with Shift).
