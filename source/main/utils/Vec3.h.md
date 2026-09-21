# source/main/utils/Vec3.h

> Minimal 3-float vector with arithmetic, dot, cross and length.

**Needs** — [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine) (conversion to/from the engine's vector)
**Used by** — [`physics/water/Buoyance.h`](../physics/water/Buoyance.h.md) · [`physics/water/Wavefield.h`](../physics/water/Wavefield.h.md)
**Tier floor** — T2: must be an unboxed value type

## Purpose

A plain value vector used in hot physics code paths because the engine's own vector type is slow when compiler optimisation is off (debug builds). The design point to keep is: **a flat struct of three single-precision floats, passed by value, no heap**.

## State

```text
RECORD Vec3
  x, y, z : real (32-bit float)
```

## Operations

**Contract** — `+`, `−`, unary `−`, scale by float (either side), divide by float, in-place variants, `dot`, `cross` (right-handed: `(y·bz − z·by, z·bx − x·bz, x·by − y·bx)`), `length`, `squared_length`, and lossless conversion to/from the engine vector.
