# source/main/gui/panels/GUI_GameMainMenu.h

> The main menu (at startup) and the pause menu (in simulation), keyboard-navigable.

**Needs** — [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_GameMainMenu.cpp`](GUI_GameMainMenu.cpp.md) · [`resources/CacheSystem.cpp`](../../resources/CacheSystem.cpp.md)
**Tier floor** — T2


## Purpose

Entry point to every other screen. Implementation: [`GUI_GameMainMenu.cpp`](GUI_GameMainMenu.cpp.md).

## State

```text
CONSTANTS: width 200, window bg (.1,.1,.1,.8), button bg (.25,.25,.24,.6), button padding 4×6
RECORD GameMainMenu: visible, button count, keyboard focus index (−1 none), enter-pressed index, title, "cache updated" notice flag
```

## API

`IsVisible`, `SetVisible` (resets keyboard focus), `Draw`, `CacheUpdatedNotice`.
