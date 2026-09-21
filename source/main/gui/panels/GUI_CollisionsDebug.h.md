# source/main/gui/panels/GUI_CollisionsDebug.h

> Debug visualiser for the terrain static collision: event boxes, collision meshes and lookup-grid cells.

**Needs** — [`physics/collision/Collisions.h`](../../physics/collision/Collisions.h.md) · [`ForwardDeclarations.h`](../../ForwardDeclarations.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_CollisionsDebug.cpp`](GUI_CollisionsDebug.cpp.md)
**Tier floor** — T2


## Purpose

Terrain authors see what the physics sees. Implementation: [`GUI_CollisionsDebug.cpp`](GUI_CollisionsDebug.cpp.md). Data comes from [`physics/collision/Collisions`](../../physics/collision/Collisions.h.md).

## State

```text
CONSTANTS: event box colour (181,51,64), collision mesh colour (209,109,44); default draw distance 200 m
RECORD CollisionsDebug
  per layer (event boxes, collision meshes, grid cells): scene nodes, enabled, draw distance (0 = unlimited)
  grid root; cell generation radius around the character (50 m)
  labels on, label shows type, label shows source; visible; hovered
```

## API

`SetVisible` (hiding turns every layer off), `IsVisible`, `IsHovered`, `Draw`, `CleanUp` (destroy all debug geometry).
