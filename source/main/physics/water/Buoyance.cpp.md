# source/main/physics/water/Buoyance.cpp

> Pressure prism volume, quadratic hull drag, partial-submersion clipping, splash emission.

**Needs** — [`Buoyance.h`](Buoyance.h.md) · [`Application.h`](../../Application.h.md) · [`SimData.h`](../SimData.h.md) · [`ActorManager.h`](../ActorManager.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md) · [`gfx/DustPool.h`](../../gfx/DustPool.h.md) · [`terrain/Terrain.h`](../../terrain/Terrain.h.md) · [`gfx/GfxWater.h`](../../gfx/GfxWater.h.md)
**Used by** — callers of [`Buoyance.h`](Buoyance.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Buoyancy is the weight of displaced water computed as the volume of the "pressure prism" over each submerged triangle; drag is a flat-plate law on the triangle normal.

## State

See [`Buoyance.h`](Buoyance.h.md).

## `computeNodeForce(a, b, c, type, t)`

```text
IF all three vertices are above the wave surface: RETURN
m = centroid; mab, mbc, mca = edge midpoints; v = mean vertex velocity
a.forces += F(a, mab, m) + F(a, m, mca)          # each vertex gets the two sub-triangles adjacent to it
b.forces += F(b, mbc, m) + F(b, m, mab)
c.forces += F(c, mca, m) + F(c, m, mbc)
```

## `computePressureForce(a, b, c, v, type)` — clipping

Water height `h` at the triangle centroid. Fully above → 0. Fully below → `sub(a, b, c)`. One vertex below → the small triangle from it to the two edge crossings. Two below → the quad split into two triangles. Crossings are linear interpolations to height h along each edge.

## `sub(a, b, c, v, type)` — submerged triangle

```text
n = (b − a) × (c − a); area = |n|/2; IF |n| < 1e-5: RETURN 0; n = unit(n)
vol = 0
IF type ≠ DRAGONLY
  a' = a + (wave_h(a) − a.y)·9810·n    (same for b', c')         # prism height = depth × ρg
  o = centroid of the six points
  vol = Σ tetra(o, faces of the prism a b c / a' b' c')          # signed volumes, tetra = (a−o)·((b−o)×(c−o))/6
drag = 0
IF type ≠ DRAGLESS
  v_rel = v − wave_velocity(centroid); s = |v_rel|
  IF s > 0.01
    cosθ = |n·v_rel/s|
    drag = −500·area·s²·cosθ·n, flipped if n·v_rel < 0            # always opposes motion along the normal
    IF update AND s·cosθ·area > 1.5: emit a splash at the first vertex within 0.1 m of the surface
IF sink: RETURN drag
IF update AND debug view: record the sub-triangle
RETURN vol·n + drag
```

**Notes** — `vol` already has units of force (depth × 9810 N/m³ × area), so it is added directly; the orientation of `n` (cab winding) decides whether it pushes outward, which is why cabs must be wound consistently.
