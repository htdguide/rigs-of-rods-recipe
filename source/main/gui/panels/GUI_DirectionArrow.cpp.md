# source/main/gui/panels/GUI_DirectionArrow.cpp

> Places an arrow mesh in the overlay layer and turns it toward the checkpoint.

**Needs** — [`GUI_DirectionArrow.h`](GUI_DirectionArrow.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`AppContext.h`](../../AppContext.h.md) · [`gfx/GfxActor.h`](../../gfx/GfxActor.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_DirectionArrow.h`](GUI_DirectionArrow.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_DirectionArrow.h`](GUI_DirectionArrow.h.md).

## State

See header.

## `CreateArrow`

Mesh `arrow2.mesh`, drawn in the overlay render queue, scale 0.1, placed at (−0.6, +0.4, −1) in view space (upper left), yaw axis fixed to world up, attached to the overlay as a 3D element; hidden.

## `Update`

When the snapshot says the arrow is visible: show and turn the node to look at the target in world space (up = +Y). Otherwise hide.
