# source/main/gui/panels/GUI_FrictionSettings.h

> Live editor for the terrain ground models (friction and fluid parameters).

**Needs** — [`Application.h`](../../Application.h.md) · [`physics/SimData.h`](../../physics/SimData.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`GameContext.cpp`](../../GameContext.cpp.md) · [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_FrictionSettings.cpp`](GUI_FrictionSettings.cpp.md) · [`main.cpp`](../../main.cpp.md)
**Tier floor** — T2


## Purpose

Tuning tool for terrain and vehicle creators. Implementation: [`GUI_FrictionSettings.cpp`](GUI_FrictionSettings.cpp.md).

## State

```text
RECORD FrictionSettings
  entries : list<{backup copy, working copy, live pointer}> — one per ground model of the terrain
  selected entry; nearest ground model (the one the player vehicle touches); visible; hovered
```

## API

`SetVisible`, `IsVisible`, `IsHovered`, `AnalyzeTerrain` (rebuild entries), `setActiveCol(model)`, `Draw`.
