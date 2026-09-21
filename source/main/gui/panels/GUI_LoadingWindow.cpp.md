# source/main/gui/panels/GUI_LoadingWindow.cpp

> Progress reporting that can render a frame from inside blocking work.

**Needs** — [`GUI_LoadingWindow.h`](GUI_LoadingWindow.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui) · [`utils/Language.h`](../../utils/Language.h.md)
**Used by** — callers of [`GUI_LoadingWindow.h`](GUI_LoadingWindow.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_LoadingWindow.h`](GUI_LoadingWindow.h.md).

## State

See header.

## `SetProgress(percent, text, render)`

Makes the window visible, stores trimmed text and counts its lines. If rendering is requested and more than 10 ms passed since the last forced frame, starts a GUI frame, draws, and renders one full frame immediately — this is what keeps the window alive while the main thread is busy loading. Every call is also logged (`<spinner>`, `<N%>`, or bare text).

## `SetProgressNetConnect(status)`

Spinner with "Joining [host:port]" and the status line.

## `Draw`

500 px wide, centred, non-interactive, height fitted to the text plus a blank line and a status row; below the text a spinner (10 blue dots), nothing, or a progress bar.
