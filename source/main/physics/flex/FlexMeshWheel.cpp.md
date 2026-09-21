# source/main/physics/flex/FlexMeshWheel.cpp

> Places the rim and rebuilds the tyre ring from tread nodes each frame.

**Needs** — [`FlexMeshWheel.h`](FlexMeshWheel.h.md) · [`Application.h`](../../Application.h.md) · [`SimData.h`](../SimData.h.md) · [`gfx/GfxActor.h`](../../gfx/GfxActor.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md)
**Used by** — callers of [`FlexMeshWheel.h`](FlexMeshWheel.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

The tyre is a 6-vertex cross-section swept around the wheel; the rim is a static mesh posed by the axis.

## State

See [`FlexMeshWheel.h`](FlexMeshWheel.h.md).

## Tyre cross-section per ray i (outer tread node o, inner tread node n)

```text
v0 = axis0 + rim_radius·unit(project(o − axis0) onto plane ⟂ axis) − centre      # rim edge, outer
v1 = o − 0.05·(o − axis0)          v2 = o − 0.1·(o − n)
v3 = n − 0.1·(n − o)               v4 = n − 0.05·(n − axis1)
v5 = axis1 + rim_radius·unit(project(n − axis1)) − centre                      # rim edge, inner
normals: v0 = axis, v5 = −axis, v2/v3 = radial, v1/v4 from neighbouring cross-section edges
V texture coordinates 0, 0.23, 0.27, 0.73, 0.77, 1; U = i/rays; the ray-0 cross-section is duplicated at the end
```

## `flexitPrepare`

Rim node at the axis midpoint, oriented with X = ±unit(axis0 − axis1) (reversed for right-side wheels), Y = unit(axis × ray to first tread node), Z = X × Y.
