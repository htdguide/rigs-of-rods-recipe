# source/main/resources/tobj_fileformat/TObjFileFormat.h

> The `.tobj` terrain-objects list: placed objects, pre-spawned vehicles, roads, trees and grass.

**Needs** — [`physics/collision/Collisions.h`](../../physics/collision/Collisions.h.md) · [`ForwardDeclarations.h`](../../ForwardDeclarations.h.md) · [Seam: 3D rendering engine](../../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine)
**Used by** — [`TObjFileFormat.cpp`](TObjFileFormat.cpp.md) · [`terrain/TerrainEditor.cpp`](../../terrain/TerrainEditor.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../../terrain/TerrainObjectManager.cpp.md)
**Tier floor** — T4

## Purpose

A terrain's `.terrn2` lists one or more `.tobj` files; each places things in the world. Read by [`TObjFileFormat.cpp`](TObjFileFormat.cpp.md), written back by the terrain editor via `TObj::WriteToStream`.

## State

```text
CONST STR_LEN = 300, LINE_BUF_LEN = 4000

RECORD TObjTree
  yaw_from, yaw_to (deg), scale_from, scale_to, high_density (1.0),
  min_distance (90), max_distance (700), grid_spacing (0)
  tree_mesh, color_map, density_map, collision_mesh : text

RECORD TObjGrass              # 'grass' and 'grass2'
  range (80), technique (1 = cross quads; 0 quads, 2 sprites), grow_technique (0),
  sway_speed (0.5), sway_length (0.05), sway_distrib (10), density,
  min_x (0.2), min_y (0.2), max_x (1.0), max_y (0.6), min_h (-9999), max_h (+9999)
  material_name, color_map_filename, density_map_filename : text

RECORD TObjVehicle            # a vehicle placed by the terrain
  position, rotation (quaternion), tobj_rotation (original degrees),
  name (file), type (TRUCK, LOAD, MACHINE, BOAT, TRUCK2), comments

RECORD TObjEntry              # one object line
  position, rotation (degrees), special (TObjSpecialObject),
  odef_name ("generic" default), type, instance_name,
  rendering_distance (0 = always rendered), comments
  is_road  = special in ROAD .. ROAD_BRIDGE
  is_actor = special in {TRUCK, LOAD, MACHINE, BOAT, TRUCK2}

RECORD TObjDocument
  document_name, grid_position, grid_enabled, rot_yxz : bool
  trees, grass, vehicles, objects : lists
  proc_objects : list<ProceduralObject>        # roads, see terrain/ProceduralManager
```

## `TObjParser`

**Contract** — `Prepare()`, `ProcessLine(line)`, `ProcessOgreStream(stream)`, `Finalize()`, static `CalcRotation(degrees, rot_yxz)`. See `.cpp` twin.

## `TObj::WriteToStream(document, stream)`

**Contract** — serialises the document; see `.cpp` twin.
