# source/main/gameplay/RaceSystem.h

> Race timer and direction-arrow state used by race scripts and the HUD.

**Needs** — nothing in this repository (only standard or third-party headers)
**Used by** — [`GameContext.h`](../GameContext.h.md) · [`RaceSystem.cpp`](RaceSystem.cpp.md)
**Tier floor** — T2


## Purpose

The native half of the race system: scripts (terrain race scripts) drive it; the HUD reads it. While a race runs, the simulation pace controls are locked to 1× (see [`ActorManager`](../physics/ActorManager.cpp.md#updateinputeventsdt)). Implementation: [`RaceSystem.cpp`](RaceSystem.cpp.md).

## State

```text
RECORD RaceSystem
  race_id = −1 (none); start_time (simulation seconds); time_diff; best_time
  arrow: visible, text, target position
```

## API

`StartRaceTimer(id)`, `StopRaceTimer`, `IsRaceInProgress`, `GetRaceId`, `GetRaceTime`, `ResetRaceUI`, time-diff and best-time setters/getters, `UpdateDirectionArrow(text?, position)`, arrow getters.
