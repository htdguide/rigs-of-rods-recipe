# source/main/gameplay/Landusemap.h

> A per-metre map of ground models for the terrain, from a colour-coded image.

**Needs** — [`Application.h`](../Application.h.md) · [`physics/SimData.h`](../physics/SimData.h.md)
**Used by** — [`Landusemap.cpp`](Landusemap.cpp.md) · [`physics/collision/Collisions.cpp`](../physics/collision/Collisions.cpp.md)
**Tier floor** — T2


## Purpose

Lets terrains vary surface friction (grass, mud, asphalt…) across the heightfield. Consulted by [`groundCollision`](../physics/collision/Collisions.cpp.md#groundcollisionnode-dt). Implementation: [`Landusemap.cpp`](Landusemap.cpp.md). Available only when built with the vegetation-paging seam (its image loader is used).

## State

```text
RECORD Landusemap
  data : array[map_x × map_z] of ground_model?    # one cell per metre
  default_model : ground_model?
  map_size : Vec3 (terrain size)
```

## API

`Landusemap(config)`, `getGroundModelAt(x, z) → model?` (outside the map → default; not built → none), `loadConfig(file)`.
