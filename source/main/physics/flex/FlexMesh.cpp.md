# source/main/physics/flex/FlexMesh.cpp

> Topology and per-frame update of generated wheel meshes.

**Needs** — [`FlexMesh.h`](FlexMesh.h.md) · [`ApproxMath.h`](../ApproxMath.h.md) · [`SimData.h`](../SimData.h.md) · [`gfx/GfxActor.h`](../../gfx/GfxActor.h.md)
**Used by** — callers of [`FlexMesh.h`](FlexMesh.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Draws a wheel as flat discs plus a band, textured with a face texture (circle mapped into the unit square) and a band texture (alternating u = 0/1 around the tread).

## State

See [`FlexMesh.h`](FlexMesh.h.md).

## Topology

- Vertex count `4·rays + 2` (+ `2·rays` when rimmed).
- Face UVs on a circle of radius 0.5 (× rim ratio for the inner ring when rimmed; outer ring offset by half a ray); axis vertices at (0.5, 0.5).
- Band UVs alternate u = 0 / 1 by ray; with an odd ray count the last ray uses u = 0.5 so the seam stretches over two quads.
- Faces: a fan from each axis vertex (reverse winding on the inner side); rimmed wheels add the annulus between rim and tyre rings (3× face indices). Band: two triangles per ray.

## `updateVertices` (compute phase)

`centre = midpoint(axis1, axis2)`; every vertex = node − centre; face normals ±unit(axis1 − axis2); band normals = unit(position) (radial). Rimmed: band vertices take the outer ring's positions. `flexitFinal` uploads and returns the centre.
