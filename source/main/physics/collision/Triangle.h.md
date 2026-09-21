# source/main/physics/collision/Triangle.h

> A triangle in space with its two span vectors and a lazily computed unit normal.

**Needs** — nothing in this repository (only standard or third-party headers)
**Used by** — [`CartesianToTriangleTransform.h`](CartesianToTriangleTransform.h.md) · [`DynamicCollisions.cpp`](DynamicCollisions.cpp.md)
**Tier floor** — T2


## Purpose

A value type used by actor-vs-actor cab collision. Split out so the barycentric transform can be defined on it.

## State

```text
RECORD Triangle
  a, b, c : Vec3                  # vertices
  u = a - c, v = b - c            # span vectors (not unit length)
  normal  : Vec3 (cached)         # unit(u × v), computed on first request
```

## `normal`

**Contract** — returns the unit normal `unit(u × v)`; computed once and cached. The cache makes the object unsafe to share between threads; every user creates its own triangle per test.
