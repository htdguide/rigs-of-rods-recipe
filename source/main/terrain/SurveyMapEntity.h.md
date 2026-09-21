# source/main/terrain/SurveyMapEntity.h

> An icon on the survey (overview) map.

**Needs** — nothing in this repository (only standard or third-party headers)
**Used by** — [`gameplay/Character.h`](../gameplay/Character.h.md) · [`gfx/GfxActor.h`](../gfx/GfxActor.h.md) · [`gui/panels/GUI_SurveyMap.h`](../gui/panels/GUI_SurveyMap.h.md) · [`Terrain.h`](Terrain.h.md) · [`TerrainObjectManager.h`](TerrainObjectManager.h.md)
**Tier floor** — T2


## Purpose

Terrain objects, race checkpoints, telepoints and scripts put icons on the map; the survey-map UI draws them. Plain data.

## State

```text
RECORD SurveyMapEntity
  type (informational), caption, icon filename, resource group ("" = textures group)
  position (m), rotation (yaw, rad), id (race id ≥ 0, −1 none, other negatives for custom groups)
  cached icon texture, draw_caption = false (tooltip instead), caption colour = white
```
