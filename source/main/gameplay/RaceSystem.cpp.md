# source/main/gameplay/RaceSystem.cpp

> Race timing based on total simulation time; direction arrow show/hide.

**Needs** — [`RaceSystem.h`](RaceSystem.h.md) · [`AppContext.h`](../AppContext.h.md) · [`GameContext.h`](../GameContext.h.md)
**Used by** — callers of [`RaceSystem.h`](RaceSystem.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`RaceSystem.h`](RaceSystem.h.md).

## State

See header.

## Behaviour

- `StartRaceTimer(id)` — start = actor manager's total simulated time; diff = 0; race id set. Timing uses *simulation* time, so pausing or slow motion cannot cheat the clock.
- `StopRaceTimer` — start = 0, id = −1. `GetRaceTime` = total time − start.
- `UpdateDirectionArrow(text, pos)` — no text hides the arrow and clears the target; otherwise show with text and target.
- `ResetRaceUI` — stop timer and hide the arrow.
