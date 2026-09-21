# source/main/physics/SlideNode.cpp

> Nearest-point projection onto a rail and the corrective spring that keeps a slide node on it.

**Needs** — [`SlideNode.h`](SlideNode.h.md) · [`Actor.h`](Actor.h.md) · [`Application.h`](../Application.h.md) · [`SimData.h`](SimData.h.md)
**Used by** — callers of [`SlideNode.h`](SlideNode.h.md) (see its Used by)
**Tier floor** — T2

## Purpose

The slide-node law. It runs inside the physics step (via [`ActorSlideNode.cpp`](ActorSlideNode.cpp.md)), so it must not allocate.

## State

See [`SlideNode.h`](SlideNode.h.md).

## `NearestPointOnLine(p1, p2, t)` / `getLenTo`

**Contract** — the point on segment p1–p2 closest to t (projection clamped to the segment). `getLenTo(beam | segment | group, point)` is the distance from point to that nearest point; a missing object yields +∞. For a group it measures only the **first** segment.

## `RailGroup.FindClosestSegment(point)`

**Contract** — linear scan of all segments, returns the one at minimum distance (first wins ties).

## `RailSegment.CheckCurSlideSegment(point)`

**Contract** — returns whichever of {this, prev, next} is closest to the point. The node can therefore move at most one segment per step, which is what makes sliding along a curved rail continuous and cheap.

## `UpdatePosition`

```text
IF no beam OR beam broken: ideal = node position; RETURN
current_segment = current_segment.check_cur_slide_segment(node.pos); beam = current_segment.beam
b = beam.p2 - beam.p1; len_b = |b|; b = unit(b)
s = clamp((node.pos - beam.p1)·b, 0, len_b)
ideal = beam.p1 + b·s
forces_ratio = s / len_b   (0 if the beam has zero length)
```

## `UpdateForces(dt)`

```text
IF no beam OR beam broken OR slide node broken: RETURN
IF current_threshold > initial_threshold: current_threshold -= attach_rate·dt     # still attaching
d = ideal - node.pos
pull = -spring_rate · max(0, |d| - current_threshold)
F = unit(d) · pull
IF |F| > break_force: broken = true          # force is still applied this step
node.forces        -= F
beam.p1.forces     += F·(1 - forces_ratio)
beam.p2.forces     += F·forces_ratio
```

**Notes** — the reaction is shared by the two rail-end nodes in proportion to position, so momentum is conserved between the slider and the rail's owner (which may be another actor).

## `ResetPositions`

**Contract** — with a current rail: pick the closest segment of the whole rail, take its beam, update position. Without a rail: nothing changes.
