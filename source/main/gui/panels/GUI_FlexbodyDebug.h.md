# source/main/gui/panels/GUI_FlexbodyDebug.h

> Debug view for a vehicle flexbodies and props: base nodes, forset nodes, vertex locators and memory order.

**Needs** — [`Application.h`](../../Application.h.md) · [`physics/SimData.h`](../../physics/SimData.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_FlexbodyDebug.cpp`](GUI_FlexbodyDebug.cpp.md)
**Tier floor** — T2


## Purpose

Helps diagnose deformed-mesh attachment (see [`physics/flex/FlexBody`](../../physics/flex/FlexBody.h.md)). Implementation: [`GUI_FlexbodyDebug.cpp`](GUI_FlexbodyDebug.cpp.md).

## State

```text
RECORD FlexbodyDebug
  wireframe, show base nodes, show forset nodes, show vertices, hide other elements
  per-vertex "show locator" flags; hovered vertex
  combo items (flexbodies first, then props), index of first prop (−1 none), selection; visible; hovered
```

## API

`IsVisible`, `IsHovered`, `SetVisible`, `Draw`, `IsHideOtherElementsModeActive`, `AnalyzeFlexbodies` (rebuild the list for the player vehicle).
