# source/main/physics/collision/PointColDetector.cpp

> Refresh of candidate points each step and a lazily built kd-tree range query.

**Needs** — [`PointColDetector.h`](PointColDetector.h.md) · [`Actor.h`](../Actor.h.md) · [`ActorManager.h`](../ActorManager.h.md) · [`GameContext.h`](../../GameContext.h.md)
**Used by** — callers of [`PointColDetector.h`](PointColDetector.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Makes cab-vs-node collision sub-quadratic: each collision triangle asks only for points inside its (enlarged) bounding box.

## State

See [`PointColDetector.h`](PointColDetector.h.md).

## `UpdateIntraPoint(contactables)`

**Contract** — the point set is the owner's contacters (or all contactable nodes when asked). If the count changed, rebuild the point list; otherwise refresh cached positions. Then mark the kd-tree root as unbuilt over all points.

## `UpdateInterPoint(ignore_state)`

```text
partners = []; count = 0
FOR EACH other actor A (physics running, or any when ignore_state) whose bounding box overlaps owner's
  partners += A
  count += A is linked to owner ? A.contacters : A.contactable_nodes
  IF |owner.node0.velocity − A.node0.velocity|² > 16         # closing faster than 4 m/s
    reset the cab-check skip counters of both actors (check every step)
owner.collision_relevant = count > 0
IF partners or count changed: rebuild point list ELSE refresh positions
mark kd-tree root unbuilt
```

Point list rebuild: for each partner, include node i if it is a contacter, or if it is contactable and the partner is neither the owner nor linked to it (linked actors only collide through contacters, so a truck does not collide with its own trailer's whole body).

**Notes** — refreshing positions scans every point for every actor (O(points × actors)); the original notes it loops actors first to avoid lookups.

## `query(v1, v2, v3, enlarge)`

**Contract** — collects every indexed point inside the axis-aligned box of the three vertices grown by `enlarge` on all sides into `hit_list` / `hit_actors`.

```text
FUNCTION query_rec(k, axis)                 # iterative on the right child, recursive on the left
  LOOP
    IF node k unbuilt: build(k, axis)
    IF node k is a leaf: IF its point is inside the box: record hit; RETURN
    IF box.max[axis] >= k.middle
      IF box.min[axis] > k.max: RETURN
      IF box.min[axis] <= k.middle: query_rec(2k+1, next(axis))
      k = 2k+2; axis = next(axis)
    ELSE
      IF box.max[axis] < k.min: RETURN
      k = 2k+1; axis = next(axis)

FUNCTION build(k, axis)                     # partition refs[begin..end) on 'axis'
  n = end - begin
  n == 1: leaf with that point (min = max = middle = coordinate)
  n == 2: order the two points, min/max/middle = their coordinates, both children become leaves on next axis
  else: median = begin + n/2; quick-select partition around the median (Hoare-style),
        min = smallest coordinate left of median, max = largest right of it, middle = median coordinate,
        children [begin, median) and [median, end) marked unbuilt
```

Axes cycle x → y → z. Only the parts of the tree a query actually visits are ever built, which is why each step's rebuild is cheap.
