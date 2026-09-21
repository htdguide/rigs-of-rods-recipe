# source/main/gui/panels/GUI_GameAbout.h

> The About window: versions and credits.

**Needs** —  · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_GameAbout.cpp`](GUI_GameAbout.cpp.md)
**Tier floor** — T2


## Purpose

Informational. Implementation: [`GUI_GameAbout.cpp`](GUI_GameAbout.cpp.md).

## State

```text
RECORD GameAbout: visible
```

## API

`SetVisible` (closing in the main menu reopens the main menu), `IsVisible`, `Draw`.
