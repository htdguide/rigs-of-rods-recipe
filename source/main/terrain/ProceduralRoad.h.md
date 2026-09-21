# source/main/terrain/ProceduralRoad.h

> The generated mesh and collision of one procedural road: an 8-point cross-section extruded between consecutive blocks.

**Needs** — [`Application.h`](../Application.h.md) · [`utils/memory/RefCountingObject.h`](../utils/memory/RefCountingObject.h.md)
**Used by** — [`resources/tobj_fileformat/TObjFileFormat.cpp`](../resources/tobj_fileformat/TObjFileFormat.cpp.md) · [`scripting/bindings/ProceduralRoadAngelscript.cpp`](../scripting/bindings/ProceduralRoadAngelscript.cpp.md) · [`ProceduralManager.cpp`](ProceduralManager.cpp.md) · [`ProceduralManager.h`](ProceduralManager.h.md) · [`ProceduralRoad.cpp`](ProceduralRoad.cpp.md) · [`TerrainObjectManager.cpp`](TerrainObjectManager.cpp.md)
**Tier floor** — T2


## Purpose

Implementation: [`ProceduralRoad.cpp`](ProceduralRoad.cpp.md).

## State

```text
ENUM RoadType   = AUTOMATIC | FLAT | LEFT | RIGHT | BOTH | BRIDGE | MONORAIL
ENUM TextureFit = NONE | BRICKWALL | ROADS1 | ROADS2 | ROAD | ROADS3 | ROADS4 | CONCRETEWALL | CONCRETEWALLI | CONCRETETOP | CONCRETEUNDER
RECORD ProceduralRoad
  vertices[≤ 50000], uv[≤ 50000], triangles[≤ 50000] (16-bit indices), counts
  previous block: position, rotation, type, width, bwidth, bheight; first flag
  id (global counter), collision flag, registered collision triangle ids, mesh, scene node
```

## API

`addBlock(pos, rot, type, width, bwidth, bheight, pillartype = 1)`, `addQuad(p1..p4, texfit, pos, lastpos, width, flip)`, `addCollisionQuad(p1..p4, ground model | name, flip)`, `createMesh`, `finish(parent node)`, `setCollisionEnabled`.
