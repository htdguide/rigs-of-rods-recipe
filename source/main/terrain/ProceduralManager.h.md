# source/main/terrain/ProceduralManager.h

> Procedural road objects (point lists) and the manager that turns them into road meshes.

**Needs** — [`Application.h`](../Application.h.md) · [`ProceduralRoad.h`](ProceduralRoad.h.md)
**Used by** — [`scripting/bindings/ProceduralRoadAngelscript.cpp`](../scripting/bindings/ProceduralRoadAngelscript.cpp.md) · [`ProceduralManager.cpp`](ProceduralManager.cpp.md) · [`TerrainObjectManager.h`](TerrainObjectManager.h.md)
**Tier floor** — T2


## Purpose

Roads come from `.tobj` `begin_procedural_roads` blocks or from scripts, which can edit points and rebuild. Implementation: [`ProceduralManager.cpp`](ProceduralManager.cpp.md); geometry in [`ProceduralRoad`](ProceduralRoad.cpp.md).

## State

```text
RECORD ProceduralPoint = { position; rotation (quaternion); type : RoadType; width; bwidth (border); bheight (border height);
                           pillartype : 0 none | 1 bridge | 2 monorail; comments }
RECORD ProceduralObject = { name; points : list<Point>; road : ProceduralRoad?; smoothing_num_splits = 0; collision_enabled = true }
RECORD ProceduralManager = { objects : list<Object>; grouping scene node }
```

## API

Object: `addPoint`, `getPoint(i)` (none if out of range), `insertPoint(i, p)` / `deletePoint(i)` (ignored if out of range; insert cannot append), `getNumPoints`, `getRoad`, name. Manager: `addObject` (builds its mesh, then lists it), `removeObject` (unlists without destroying the mesh), `getNumObjects`, `getObject(i)`, `rebuildObjectMesh`, `deleteObjectMesh`, `removeAllObjects`, `logDiagnostics`.
