# source/main/physics/collision/PointColDetector.h

> A per-actor kd-tree over collision points (own contacters, or other actors’ contactable nodes) queried by triangle bounding box.

**Needs** — [`Application.h`](../../Application.h.md)
**Used by** — [`physics/Actor.cpp`](../Actor.cpp.md) · [`physics/ActorManager.cpp`](../ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`DynamicCollisions.cpp`](DynamicCollisions.cpp.md) · [`PointColDetector.cpp`](PointColDetector.cpp.md)
**Tier floor** — T2


## Purpose

Every actor with collisions enabled owns two detectors: an *intra* detector (its own nodes, for self-collision) and an *inter* detector (nodes of nearby actors). Cab triangles query them each step. Implementation: [`PointColDetector.cpp`](PointColDetector.cpp.md).

## State

```text
RECORD PointId      = { actor_id, node_num }
RECORD RefElem      = { point_id_index, point : [3]float }      # cached node position
RECORD KdNode       = { min, max, middle : float; begin, end : int; ref : RefElem index or none }
                      # end < 0 means "not built yet"; the tree is built lazily during queries

RECORD PointColDetector
  owner              : Actor
  partners           : list<actor id>        # intra: [owner]; inter: overlapping actors this step
  refs               : list<RefElem>
  kdtree             : list<KdNode>          # size = max(1, 2^(ceil(log2 n) + 1))
  object_count       : int = -1              # number of points currently indexed
  query_box          : (min, max)
  hit_list           : list<point id index>  # results of the last query
  hit_actors         : set<actor id>         # actors that appear in hit_list
  point_ids          : list<PointId>
```

## API

`UpdateIntraPoint(contactables?)`, `UpdateInterPoint(ignore_state?)`, `query(v1, v2, v3, enlarge)`.
