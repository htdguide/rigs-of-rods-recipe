# source/main/gui/panels/GUI_SimPerfStats.cpp

> Draws the renderer statistics as four single-bar histograms.

**Needs** — [`GUI_SimPerfStats.h`](GUI_SimPerfStats.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`AppContext.h`](../../AppContext.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_SimPerfStats.h`](GUI_SimPerfStats.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_SimPerfStats.h`](GUI_SimPerfStats.h.md).

## State

Stateless beyond visibility.

## `Draw`

Top-left, semi-transparent, non-interactive. Title `FPS: x, Batch: n, Tri: n` from the render window statistics; four 60×35 bars labelled Current, Average, Worst, Best, each scaled 0 … best FPS and captioned with two decimals.
