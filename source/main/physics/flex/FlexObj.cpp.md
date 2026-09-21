# source/main/physics/flex/FlexObj.cpp

> Maps cab triangles onto per-submesh vertices and rebuilds positions/normals each frame.

**Needs** — [`FlexObj.h`](FlexObj.h.md) · [`ApproxMath.h`](../ApproxMath.h.md) · [`gfx/GfxActor.h`](../../gfx/GfxActor.h.md)
**Used by** — callers of [`FlexObj.h`](FlexObj.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Turns a node-indexed triangle list into a textured mesh. Vertices are *texcoord entries*, not nodes: the same node may appear as several vertices (one per submesh / UV seam).

## State

See [`FlexObj.h`](FlexObj.h.md).

## Construction

For each triangle corner, find the vertex for that node **within the triangle's submesh span** of texcoords (the first matching entry; 0 if none). Each submesh gets the index range of its own triangles. Reference area per triangle = 2·|(v1−v0)×(v2−v0)| at spawn. Bounds are a fixed ±100 m box.

## `UpdateMesh` (per frame, on the node snapshot)

```text
centre = midpoint of the first two vertices' nodes
position[v] = node(v) − centre; normal[v] = 0
FOR EACH triangle
  n = (p1 − p0) × (p2 − p0); s = |n|
  IF s > ref_area: collapse the triangle (p1 = p0 + (0.1,0,0), p2 = p0 + (0,0,0.1))     # torn body panels vanish
  IF s == 0: CONTINUE
  add n/s to the three vertex normals
normalise normals (fast approximation)
RETURN centre
```

`UpdateFlexObj` also uploads the vertex buffer. `ScaleFlexObj(f)` scales the reference areas (used with `scaleTruck`).
