# source/main/gui/panels/GUI_DirectionArrow.h

> The 3D arrow in the screen corner pointing to the next race checkpoint.

**Needs** — [`ForwardDeclarations.h`](../../ForwardDeclarations.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gfx/GfxScene.cpp`](../../gfx/GfxScene.cpp.md) · [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_DirectionArrow.cpp`](GUI_DirectionArrow.cpp.md) · [`main.cpp`](../../main.cpp.md)
**Tier floor** — T2


## Purpose

Race guidance. The target and visibility are decided by the race system and delivered through the sim buffer; this panel only presents them. Implementation: [`GUI_DirectionArrow.cpp`](GUI_DirectionArrow.cpp.md).

## State

```text
RECORD DirectionArrow: arrow scene node, overlay "tracks/DirectionArrow", caption text, distance text
```

## API

`LoadOverlay` (after resources load), `CreateArrow` (again after every scene reset), `Update(player vehicle)`, `IsVisible`, `SetVisible` (menu only; in simulation `Update` decides).
