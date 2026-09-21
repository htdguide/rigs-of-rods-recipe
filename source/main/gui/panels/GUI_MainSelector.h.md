# source/main/gui/panels/GUI_MainSelector.h

> The content selector ("Loader"): browse, search and pick terrains, vehicles, skins, dashboards or gadgets from the mod cache.

**Needs** — [`Application.h`](../../Application.h.md) · [`physics/SimData.h`](../../physics/SimData.h.md) · [`resources/CacheSystem.h`](../../resources/CacheSystem.h.md) · [`ForwardDeclarations.h`](../../ForwardDeclarations.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`GameContext.cpp`](../../GameContext.cpp.md) · [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_GameMainMenu.cpp`](GUI_GameMainMenu.cpp.md) · [`GUI_MainSelector.cpp`](GUI_MainSelector.cpp.md) · [`GUI_SurveyMap.cpp`](GUI_SurveyMap.cpp.md) · [`GUI_TopMenubar.cpp`](GUI_TopMenubar.cpp.md) · [`main.cpp`](../../main.cpp.md)
**Tier floor** — T2


## Purpose

One picker for every content type, fed by [`resources/CacheSystem`](../../resources/CacheSystem.h.md) queries. Implementation: [`GUI_MainSelector.cpp`](GUI_MainSelector.cpp.md).

## State

```text
CONSTANTS: list pane 250 px; preview up to 70 % of the view
RECORD DisplayCategory: category id, title "(count) name"
RECORD DisplayEntry: cache entry; pre-formatted modified time, install time, vehicle type
RECORD MainSelector
  loader type (NONE = hidden); categories; entries; search method + string; GUID filter (skins); search input (500)
  show details; search box was active; advertised entry (always first — e.g. the default skin); hovered; keyboard focus pending
  selected category position and id; selected entry (−1 = empty list); selected section config
  per loader type: last category position, last category id, last entry
```

## API

`Show(type, guid filter, advertised entry)`, `IsVisible`, `IsHovered`, `Draw`, `Close`.
