# source/main/physics/collision/CartesianToTriangleTransform.h

> Maps a point to barycentric coordinates of its projection on a triangle plus its signed distance from the plane.

**Needs** — [`Triangle.h`](Triangle.h.md)
**Used by** — [`DynamicCollisions.cpp`](DynamicCollisions.cpp.md)
**Tier floor** — T2


## Purpose

The geometric kernel of cab collisions: one 3×3 matrix inverse per triangle, then a matrix-vector product per candidate point.

## State

```text
RECORD CartesianToTriangleTransform
  triangle : Triangle
  M⁻¹      : Mat3 (cached)        # inverse of [u v n] (columns)
```

## `apply(p) → TriangleCoord`

**Contract** — returns `(alpha, beta, gamma, distance)` where the projection of p onto the triangle plane equals `alpha·a + beta·b + gamma·c`, `alpha + beta + gamma = 1`, and `distance` is the signed distance from the plane along the unit normal.

```text
[alpha, beta, d] = M⁻¹ · (p - c)         where M = [u | v | n], u = a - c, v = b - c
gamma = 1 - alpha - beta
```

The inverse is computed on first use (lazy), so constructing a transform for a triangle that turns out to have no candidate points costs nothing.
