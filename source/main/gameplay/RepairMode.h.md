# source/main/gameplay/RepairMode.h

> Quick repair and live repair ("interactive reset") of the player’s vehicle.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`GameContext.h`](../GameContext.h.md) · [`RepairMode.cpp`](RepairMode.cpp.md)
**Tier floor** — T2


## Purpose

Holding the repair key repairs continuously; after `sim_live_repair_interval` seconds (or with the dedicated key) it becomes *live repair*, in which the driving keys move and rotate the vehicle. Owned by the game context. Implementation: [`RepairMode.cpp`](RepairMode.cpp.md).

## State

```text
RECORD RepairMode = { quick_active, live_active : bool; live_timer : s }
```

## API

`UpdateInputEvents(dt)`, `IsLiveRepairActive`, `IsQuickRepairActive`, `GetLiveRepairTimer`.
