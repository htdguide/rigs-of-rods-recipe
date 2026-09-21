# source/main/resources/odef_fileformat/ODefFileFormat.h

> The `.odef` terrain-object definition: mesh, scale, collision boxes and meshes, lights, particles, sounds, animations.

**Needs** — [`physics/SimData.h`](../../physics/SimData.h.md) (collision event filters, localizer types) · [Seam: 3D rendering engine](../../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine)
**Used by** — [`ODefFileFormat.cpp`](ODefFileFormat.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../../terrain/TerrainObjectManager.cpp.md) · [`terrain/TerrainObjectManager.h`](../../terrain/TerrainObjectManager.h.md)
**Tier floor** — T4

## Purpose

Every static object placed on a terrain (buildings, fences, signs, ramps) is an `.odef`: the visual mesh plus how it collides and what it emits. Collision boxes double as **event boxes** that fire script events when something enters them (spawn zones, checkpoints, shops). Parsing: [`ODefFileFormat.cpp`](ODefFileFormat.cpp.md).

## State

```text
RECORD ODefCollisionBox
  aabb_min, aabb_max  : (x, y, z)      # box in object space
  box_rot             : (x, y, z) deg  # extra box rotation ('rotate')
  cam_pos             : (x, y, z)      # forced camera position ('forcecamera')
  direction           : (x, y, z)      # event direction hint
  scale               : (x, y, z)      # the object's header scale
  reverb_preset_name  : text
  event_name          : text
  event_filter        : NONE | ALL | AVATAR | TRUCK | TRUCK_WHEELS | AIRPLANE | BOAT
  is_rotating, is_virtual (no physical collision, events only), force_cam_pos : bool

RECORD ODefCollisionMesh { mesh_name, scale, groundmodel_name }
RECORD ODefParticleSys   { instance_name, template_name, pos, scale }
RECORD ODefAnimation     { name, speed_min, speed_max }
RECORD ODefTexPrint      { font_name, font_size, font_dpi, text, option(char), x, y, w, h, r, g, b, a }
RECORD ODefSpotlight     { pos, dir, colour, range, angle_inner(deg), angle_outer(deg) }
RECORD ODefPointLight    { pos, dir, colour, range }

RECORD ODefDocument
  header           : { mesh_name, scale, cast_shadows = true }
  mode_standard    : bool
  localizers       : list<VOR | NDB | VERTICAL | HORIZONTAL>     # aviation beacons
  sounds           : list<sound script name>
  groundmodel_files: list<file>
  collision_boxes, collision_meshes, particle_systems, animations,
  texture_prints, spotlights, point_lights : lists
  mat_name          : text     # 'setMeshMaterial'
  mat_name_generate : text     # 'generateMaterialShaders'
```

## `ODefParser`

**Contract** — `Prepare()`, `ProcessLine(line)` (returns false at `end`), `ProcessOgreStream(stream)`, `Finalize()` → document. See `.cpp` twin.
