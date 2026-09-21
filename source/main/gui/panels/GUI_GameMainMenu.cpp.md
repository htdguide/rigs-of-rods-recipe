# source/main/gui/panels/GUI_GameMainMenu.cpp

> Menu buttons per application state, keyboard navigation, version and notice boxes.

**Needs** — [`GUI_GameMainMenu.h`](GUI_GameMainMenu.h.md) · [`Application.h`](../../Application.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [`GUI_MainSelector.h`](GUI_MainSelector.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`utils/PlatformUtils.h`](../../utils/PlatformUtils.h.md) · [`RoRVersion.h`](../../../../source/version_info/RoRVersion.h.md) · [`network/RoRnet.h`](../../network/RoRnet.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_GameMainMenu.h`](GUI_GameMainMenu.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_GameMainMenu.h`](GUI_GameMainMenu.h.md).

## State

See header.

## Buttons

| Main menu | Pause / multiplayer menu |
|---|---|
| Single player → terrain selector | Resume game → unpause |
| Resume game (only if `autosave.sav` exists) → load it | Repository |
| Multiplayer → server selector | Controls |
| Repository | Return to menu → unload terrain, disconnect if online, open menu |
| Settings | Exit game |
| Controls | |
| About | |
| Exit game → shutdown | |

Every button that opens a screen hides the menu. The title is "Main menu", "Pause" or (online) "Menu".

## Placement

Bottom-left, margin = screen height / 15; centred instead on very wide, short displays (> 2200 × < 1100). Auto-sized to its buttons.

## Keyboard

When the GUI isn't capturing the keyboard (or the menu is hovered): Up/Down move a focus marker `--> text <--` with wrap-around; Enter activates the focused button on the next draw. Online in simulation the menu captures the keyboard and Escape closes it.

## Boxes (main menu only)

Bottom-right, non-interactive: game version and network protocol version; above it an "Cache updated" notice with an icon after a cache update, cleared when the menu hides.
