# source/main/gui/panels/GUI_LoadingWindow.h

> The modal "Please wait" window with text and either a progress bar or a spinner.

**Needs** — [`Application.h`](../../Application.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_GameControls.cpp`](GUI_GameControls.cpp.md) · [`GUI_LoadingWindow.cpp`](GUI_LoadingWindow.cpp.md) · [`GUI_MainSelector.cpp`](GUI_MainSelector.cpp.md) · [`main.cpp`](../../main.cpp.md) · [`resources/CacheSystem.cpp`](../../resources/CacheSystem.cpp.md) · [`terrain/Terrain.cpp`](../../terrain/Terrain.cpp.md) · [`terrain/TerrainGeometryManager.cpp`](../../terrain/TerrainGeometryManager.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../../terrain/TerrainObjectManager.cpp.md)
**Tier floor** — T2


## Purpose

Feedback during blocking work (terrain load, cache update, connecting). Implementation: [`GUI_LoadingWindow.cpp`](GUI_LoadingWindow.cpp.md).

## State

```text
CONSTANTS: HIDE_PROGRESSBAR = −1, SHOW_SPINNER = −2
RECORD LoadingWindow: percent, visible, text, text line count, frame timer, spinner counter
```

## API

`SetProgress(percent, text, render frame = true)`, `SetProgressNetConnect(status)`, `Draw`, `SetVisible`, `IsVisible`.
