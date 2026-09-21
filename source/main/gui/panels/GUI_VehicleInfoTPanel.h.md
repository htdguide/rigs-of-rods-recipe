# source/main/gui/panels/GUI_VehicleInfoTPanel.h

> The left-side vehicle panel: quick-action buttons, live statistics, command key list, and debug views.

**Needs** — [`ForwardDeclarations.h`](../../ForwardDeclarations.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_VehicleInfoTPanel.cpp`](GUI_VehicleInfoTPanel.cpp.md) · [`main.cpp`](../../main.cpp.md)
**Tier floor** — T2


## Purpose

Everything the player needs about the current vehicle in one place; appears translucent when the mouse nears the left edge, opaque when summoned by hotkey. Implementation: [`GUI_VehicleInfoTPanel.cpp`](GUI_VehicleInfoTPanel.cpp.md).

## State

```text
ENUM TPanelMode: HIDDEN, OPAQUE, TRANSLUCENT
ENUM TPanelFocus: NONE, BASICS, COMMANDS, STATS, DIAG
RECORD VehicleInfoTPanel
  mode; requested tab; current tab; startup hint timer + shown flag
  translucent bg (.1,.1,.1,.5); translucent disabled-text colour
  active command key (held via the panel), hovered command key; command beam highlight colour (.733,1,.157,.745), thickness 15
  help image full-size flag + screen position
  stats: health, broken beams, deformed beams, summed stress, mass kg, summed deformation, current and max g (3 axes)
  horn button held; icons
CONSTANTS: help image 512×80 (full 512×128); preview ≤100×100; panel min width 230
```

## API

`SetVisible(mode, tab)`, `IsVisible(tab)` (opaque and, if given, on that tab), `GetActiveCommandKey` (read by the actor input code so clicking a key button drives that command), `IsHornButtonActive`, `UpdateStats(dt, actor)` (reads live beams — sim thread synced), `Draw(gfx actor)` (from the snapshot).
