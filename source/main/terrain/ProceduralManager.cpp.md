# source/main/terrain/ProceduralManager.cpp

> Building road meshes from points, with optional spline smoothing.

**Needs** — [`ProceduralManager.h`](ProceduralManager.h.md) · [`Application.h`](../Application.h.md) · [`ProceduralRoad.h`](ProceduralRoad.h.md)
**Used by** — callers of [`ProceduralManager.h`](ProceduralManager.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`ProceduralManager.h`](ProceduralManager.h.md).

## State

See header.

## `rebuildObjectMesh(object)`

```text
drop the old road; road = new ProceduralRoad(collision flag)
IF smoothing: spline through point positions (Catmull-Rom tangents)
FOR EACH point i
  IF smoothing
    FOR s IN 1..splits+1: t = s/(splits+1)
      i == 0: add block at the first point unchanged          # repeated splits+1 times (original behaviour)
      else: add block at spline(segment i−1, t) with rotation nlerp(prev, cur, t), width/bwidth/bheight lerped, current type and pillar type
  ELSE add block with the point's values
road.finish(grouping node)
```

`logDiagnostics` writes every object's points with type and pillar legends to the log.

**Notes** — the first point is emitted `splits + 1` times when smoothing; the road generator treats repeated identical blocks as zero-length segments.
