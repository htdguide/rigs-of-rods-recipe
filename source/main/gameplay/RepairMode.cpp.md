# source/main/gameplay/RepairMode.cpp

> Maps keys to translation/rotation requests and queues a reset every frame while repairing.

**Needs** — [`RepairMode.h`](RepairMode.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`GameContext.h`](../GameContext.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md)
**Used by** — callers of [`RepairMode.h`](RepairMode.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Repair = keep resetting the actor's structure while letting the player re-position it.

## State

See [`RepairMode.h`](RepairMode.h.md).

## `UpdateInputEvents(dt)`

```text
IF no player actor: clear flags and timer; RETURN
live key → live = true (timer primed past the interval)
quick = repair key held; IF quick AND interval > 0: live = timer > interval;  IF NOT quick: timer = 0
IF quick OR live
  translation: accelerate/brake = ±2 m/s vertical; character forward/back/sidestep = ±2 m/s along/across the heading
  rotation: steer left/right = ±0.5 rad/s
  IF any movement
    scale = (Alt ? 0.1 : 1)·(Shift ? 3 : 1)·(Ctrl ? 10 : 1); rotation ×clamp(scale, 0.1, 10); translation ×scale
    request rotation about the actor's rotation centre and translation (also for linked actors in soft-reset mode); timer = 0
  ELSE IF Space pressed: request 45° angle snap (linked actors too in soft-reset mode)
  ELSE timer += dt
  queue a reset request for the actor (and, in soft-reset mode, each linked actor): SOFT_RESET if sim_soft_reset_mode else RESET_ON_SPOT
```

The queued reset is what repairs the structure; rotation/translation requests are applied during that reset by the actor ([`HandleInputEvents`](../physics/Actor.cpp.md#resets)).
