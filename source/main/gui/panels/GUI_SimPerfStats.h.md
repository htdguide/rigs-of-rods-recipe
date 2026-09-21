# source/main/gui/panels/GUI_SimPerfStats.h

> Frame-rate overlay: current, average, worst and best FPS with batch and triangle counts.

**Needs** —  · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_SimPerfStats.cpp`](GUI_SimPerfStats.cpp.md)
**Tier floor** — T2


## Purpose

Quick performance readout (toggled by hotkey). Implementation: [`GUI_SimPerfStats.cpp`](GUI_SimPerfStats.cpp.md).

## State

```text
RECORD SimPerfStats: visible
```

## API

`SetVisible`, `IsVisible`, `Draw`.
