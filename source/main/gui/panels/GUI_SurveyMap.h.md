# source/main/gui/panels/GUI_SurveyMap.h

> The overview map: a round mini-map following the player, or a big full-terrain map, with icons, teleport and AI waypoints.

**Needs** — [`Application.h`](../../Application.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui) · [`physics/SimData.h`](../../physics/SimData.h.md) · [`terrain/SurveyMapEntity.h`](../../terrain/SurveyMapEntity.h.md) · [`gfx/SurveyMapTextureCreator.h`](../../gfx/SurveyMapTextureCreator.h.md)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_SurveyMap.cpp`](GUI_SurveyMap.cpp.md) · [`terrain/Terrain.cpp`](../../terrain/Terrain.cpp.md)
**Tier floor** — T2


## Purpose

Navigation and quick travel. The terrain image is rendered once per terrain from above. Implementation: [`GUI_SurveyMap.cpp`](GUI_SurveyMap.cpp.md). Entities come from [`terrain/SurveyMapEntity.h`](../../terrain/SurveyMapEntity.h.md).

## State

```text
ENUM SurveyMapMode: NONE, SMALL, BIG
RECORD SurveyMap
  mode, last shown mode; hovered; dragging a waypoint + which
  terrain size (m) and map offset (m); zoom 0–1; map texture
  mouse-hint icons; mini-map circle centre and radius
CONSTANTS: window padding 4, rounding 2
```

## API

`CreateTerrainTextures` (on terrain load), `Draw`, `IsVisible`, `IsHovered`, `CycleMode` (none → small → big → none), `ToggleMode` (none ↔ last shown).
