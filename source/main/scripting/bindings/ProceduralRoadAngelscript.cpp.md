# source/main/scripting/bindings/ProceduralRoadAngelscript.cpp

> Script types for building procedural roads.

**Needs** — [`terrain/ProceduralManager.h`](../../terrain/ProceduralManager.h.md) · [`terrain/ProceduralRoad.h`](../../terrain/ProceduralRoad.h.md) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## Types

- `ProceduralPointClass` (factory) — properties position, rotation, width, border_width, border_height, type (`RoadType`), pillar_type.
- `ProceduralRoadClass` (factory) — `addBlock(pos, rot, type, width, bwidth, bheight, pillartype)`, `addQuad`, `addCollisionQuad`, `createMesh`, `finish`, `setCollisionEnabled`.
- `ProceduralObjectClass` (factory) — `getName`/`setName`, `addPoint`, `insertPoint`, `deletePoint`, `getPoint`, `getNumPoints`, `getRoad`, properties smoothing_num_splits, collision_enabled.
- `ProceduralManagerClass` — `addObject`, `removeObject`, `getNumObjects`, `getObject`, `rebuildObjectMesh`, `deleteObjectMesh`.
- Enums `RoadType` (AUTOMATIC, FLAT, LEFT, RIGHT, BOTH, BRIDGE, MONORAIL) and `TextureFit` (NONE, BRICKWALL, ROADS1, ROADS2, ROAD, ROADS3, ROADS4, CONCRETEWALL, CONCRETEWALLI, CONCRETETOP, CONCRETEUNDER).

See [`terrain/ProceduralRoad`](../../terrain/ProceduralRoad.h.md) and [`terrain/ProceduralManager`](../../terrain/ProceduralManager.h.md).
