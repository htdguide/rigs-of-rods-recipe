# source/main/gameplay/SceneMouse.h

> Mouse interaction with the 3D scene: grab-and-drag nodes, toggle hooks, select vehicles and orbit centres.

**Needs** — [`Application.h`](../Application.h.md) · [`physics/SimData.h`](../physics/SimData.h.md)
**Used by** — [`GameContext.h`](../GameContext.h.md) · [`SceneMouse.cpp`](SceneMouse.cpp.md)
**Tier floor** — T2


## Purpose

Owned by the game context; receives mouse events from the input layer when no UI has focus. Implementation: [`SceneMouse.cpp`](SceneMouse.cpp.md).

## State

```text
RECORD SceneMouse
  grab_state : 0 idle | 1 grabbing; grabbed actor, node, ray distance, last grab point, last mouse x/y
  pick line (blue line from the node to the mouse point) + scene node
```

## API

`handleMouseMoved`, `handleMousePressed`, `handleMouseReleased`, `UpdateSimulation` (per frame), `InitializeVisuals`, `UpdateVisuals`, `DiscardVisuals`.
