# source/main/gui/panels/GUI_SurveyMap.cpp

> Map texture capture, map-to-world transforms, icons, and mouse actions (teleport, waypoints).

**Needs** — [`GUI_SurveyMap.h`](GUI_SurveyMap.h.md) · [`AppContext.h`](../../AppContext.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`resources/ContentManager.h`](../../resources/ContentManager.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`gfx/GfxActor.h`](../../gfx/GfxActor.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUI_MainSelector.h`](GUI_MainSelector.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [`utils/InputEngine.h`](../../utils/InputEngine.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui) · [`gfx/SurveyMapTextureCreator.h`](../../gfx/SurveyMapTextureCreator.h.md) · [`terrain/Terrain.h`](../../terrain/Terrain.h.md) · [`terrain/TerrainObjectManager.h`](../../terrain/TerrainObjectManager.h.md) · [`physics/collision/Collisions.h`](../../physics/collision/Collisions.h.md)
**Used by** — callers of [`GUI_SurveyMap.h`](GUI_SurveyMap.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_SurveyMap.h`](GUI_SurveyMap.h.md).

## State

See header.

## `CreateTerrainTextures`

Reset offset, drop the old texture, zoom 0.5, hidden. Map extent = the terrain size, unless there is no height map, or the terrain is flat with a collision box larger than 50 m that is smaller than the terrain: then use the collision box (offset to its corner). Render a 4096² top-down capture with the renderer's FSAA ([`gfx/SurveyMapTextureCreator`](../../gfx/SurveyMapTextureCreator.h.md)) and keep it as a static texture.

## `Draw`

Hidden when the big map is open in free camera, or the selector is open. Hotkey toggles icons. In small mode, zoom keys change zoom by frame time and the mouse wheel by 0.5 per notch while hovered; zoom step `0.5·Δ·(1 − zoom)`, clamped to [0, (size − 50)/size].

- **Big** — centred, height `0.55 × window width` (minus paddings), width by terrain aspect; the whole texture.
- **Small** — top-right at y = 100, square `0.2 × window width`, transparent; shows a window of `size × (1 − zoom)` centred on the player vehicle (or character), clamped inside the terrain, drawn as a circular image with a ring.

World → map: `view_pos + (world.xz − origin) / visible_size × view_size`. In small mode, anything beyond 0.8·r² of the circle is not drawn.

## Mouse

- Left click → teleport the player there (unless on a waypoint).
- Right click → add an AI waypoint at the clicked point, height from the collision surface.
- Middle click → clear all waypoints (or, on a waypoint, remove that one).
- Drag a waypoint (within 5 px) to move it.
- Hover shows a red dot; in small mode the label "Teleport/Waypoint".

Waypoints are red dots joined by lines, yellow when hovered with a tooltip (and position in big mode). A hint row shows mouse-button icons with the current meaning of each button.

## Icons (when enabled)

- Terrain entities (unless decluttering): checkpoints only for the running race, race starts hidden while another race runs.
- Every vehicle: `icon_<type>[_activated|_networked].dds` (type: load, truck, airplane, boat, machine; AI vehicles by what they have), rotated by heading; remote ones captioned with the owner's name in their colour.
- Characters on foot: `icon_person_activated.dds` or `_networked` with caption.

Icons load lazily per entity; missing files fall back to `icon_missing.dds`. Uncaptioned icons with a caption show it as a tooltip within 5 px.
