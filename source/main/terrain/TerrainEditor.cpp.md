# source/main/terrain/TerrainEditor.cpp

> Editor controls, selection, and the two save paths.

**Needs** — [`TerrainEditor.h`](TerrainEditor.h.md) · [`AppContext.h`](../AppContext.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`gfx/camera/CameraManager.h`](../gfx/camera/CameraManager.h.md) · [`system/Console.h`](../system/Console.h.md) · [`resources/ContentManager.h`](../resources/ContentManager.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md) · [Seam: Immediate-mode GUI](../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui) · [`Terrain.h`](Terrain.h.md) · [`TerrainObjectManager.h`](TerrainObjectManager.h.md) · [`resources/tobj_fileformat/TObjFileFormat.h`](../resources/tobj_fileformat/TObjFileFormat.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md)
**Used by** — callers of [`TerrainEditor.h`](TerrainEditor.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`TerrainEditor.h`](TerrainEditor.h.md).

## State

See header.

## `UpdateInputEvents(dt)`

```text
middle click: select the object whose position is closest to the mouse ray (perpendicular distance)
enter/exit key: with nothing selected select the object nearest the free camera (or the character); else deselect
respawn-last key: place another copy of the last selected object at the character (instance "Console")
next/previous keys: cycle selection (wrapping)
rescue key: cycle rotation axis Y → Z → X (console notice); Space: toggle object tracking
reset key: restore initial position and rotation
IF selected AND camera not free
  translation: accelerate/brake ±0.5 m/s in Y; forward/back ±0.5 in X; sidestep ±0.5 in Z; rotation: steer ±2 °/s about the axis
  scale (Alt 0.1, Shift 3, Ctrl 10); apply; if tracking move the character to the object
  ELSE IF tracking and the character moved: move the object to the character
  remove key: destroy the object
ELSE update characters (walk around)
toggle key: request leaving the editor
```

## Selecting

Selecting announces "Selected object: [i/n] (name)", moves the character there when tracking, and (for predefined actors) makes sure the actor exists, respawning it if necessary.

## Moving objects

Static objects move their scene node (orientation X·Y·Z then pitch −90°). Predefined actors are *soft-respawned* at the new position/rotation (tobj rotation convention) and their visuals refreshed. Collision boxes and triangles are **not** moved — only the edited `.tobj` reproduces them on reload.

## Saving (on leaving the editor)

- `WriteEditsToTobjFiles` — only for unpacked terrains ("FileSystem" bundles; else a console warning): for each cached tobj, rebuild its object and vehicle lists from the editor objects that came from it (name, instance, type, position, rotation, comments) and overwrite the file via the [TObj writer](../resources/tobj_fileformat/TObjFileFormat.cpp.md). The per-file default rendering distance is not reconstructed.
- `WriteSeparateOutputFile` — the legacy dump `editor_out.log` in the logs directory: one line per static object `x, y, z, rx, ry, rz, name` (8.3 / 6.1 formats) and each procedural road as a `begin_procedural_roads … end_procedural_roads` block with `x, y, z, 0, yaw, 0, width, bwidth, bheight, type` (type names auto, flat, left, right, both, bridge / bridge_no_pillars, monorail / monorail2).
