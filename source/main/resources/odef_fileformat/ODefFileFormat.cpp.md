# source/main/resources/odef_fileformat/ODefFileFormat.cpp

> Reads `.odef` files.

**Needs** — [`ODefFileFormat.h`](ODefFileFormat.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md)
**Used by** — callers of [`ODefFileFormat.h`](ODefFileFormat.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Defines the `.odef` grammar: a two-line header followed by keyword lines.

## State

```text
header_done, header_mesh_name, header_scale
current collision box/mesh context:
  direction, is_rotating, is_virtual, force_cam, rotation, cam_pos,
  event_filter, event_name, mesh_name, reverb_preset, groundmodel_name ("concrete"), aabb_min, aabb_max
```

## Header

```text
line: LOD            -> ignored (obsolete), may appear before the mesh line
line: <mesh name>    -> taken verbatim (not trimmed)
line: sx, sy, sz     -> scale; header complete
```

## Body lines

Each line is trimmed and UTF-8-sanitised; empty lines and lines starting with `/` or `;` are skipped; the exact line `end` stops reading. Prefix-matched keywords (`sscanf`-style comma-separated numbers):

| Line | Effect |
|---|---|
| `movable` | ignored |
| `standard` | mode_standard = true |
| `localizer-vor` / `-ndb` / `-v…` / `-h…` | add localizer (checked in that order) |
| `sound <name>` | add sound script (≤ 200 chars) |
| `particleSystem scale, x, y, z, <instance> <template>` | add particle system (all 6 needed) |
| `setMeshMaterial <name>` | material override |
| `generateMaterialShaders <name>` | material to generate shaders for |
| `playanimation min_speed, max_speed, <name>` | add looping mesh animation (name required) |
| `drawTextOnMeshTexture x, y, w, h, r, g, b, a, <c option>, font_size, font_dpi, <font> <text>` | text baked into the texture (all 13 needed); `text` is a single token |
| `spotlight px, py, pz, dx, dy, dz, r, g, b, range, inner_deg, outer_deg` | all 12 needed |
| `pointlight px, py, pz, dx, dy, dz, r, g, b, range` | all 10 needed |
| `beginbox` / `beginmesh` | reset the box/mesh context |
| `boxcoords x1, x2, y1, y2, z1, z2` | **min/max per axis** (note the order: x-min, x-max, y-min, …) |
| `mesh <name>` | collision mesh name |
| `rotate rx, ry, rz` | box rotation; marks rotating |
| `forcecamera x, y, z` | forced camera position; flag |
| `direction x, y, z` | event direction |
| `frictionconfig <file>` | extra ground-model file |
| `stdfriction <name>` / `usefriction <name>` | ground model of the collision mesh |
| `virtual` | box does not collide, events only |
| `event <name> [<type>]` | event box; type prefix `avatar`, `truck_wheels`, `truck`, `airplane`, `boat`, anything else = all. Names starting `spawnzone` are forced to `avatar` (performance hack) |
| `reverb_preset <name>` | sound reverb preset inside the box |
| `endbox` | append a collision box from the context (scale = header scale) |
| `endmesh` | append a collision mesh from the context |
| `nocast` | header.cast_shadows = false |

The context is not reset by `endbox`/`endmesh`, only by `beginbox`/`beginmesh`, so attributes carry over between boxes unless reset.
