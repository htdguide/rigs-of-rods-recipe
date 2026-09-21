# source/main/gui/panels/GUI_NodeBeamUtils.h

> Tuning window for the player vehicle soft-body: mass, spring/damping scales, and an automatic parameter search.

**Needs** — [`Application.h`](../../Application.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_NodeBeamUtils.cpp`](GUI_NodeBeamUtils.cpp.md)
**Tier floor** — T2


## Purpose

Helps vehicle authors find stable spring/damping values interactively. Implementation: [`GUI_NodeBeamUtils.cpp`](GUI_NodeBeamUtils.cpp.md). The search itself is done by the actor ([`physics/Actor`](../../physics/Actor.h.md)).

## State

```text
RECORD NodeBeamUtils: visible, hovered, searching
```

## API

`Draw`, `SetVisible` (hiding stops a search), `IsVisible`, `IsHovered`.
