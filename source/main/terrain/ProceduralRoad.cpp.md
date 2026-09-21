# source/main/terrain/ProceduralRoad.cpp

> Cross-section profiles per road type, segment quads with texture atlas mapping, bridge pillars, collision triangles.

**Needs** — [`ProceduralRoad.h`](ProceduralRoad.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`Application.h`](../Application.h.md) · [`physics/collision/Collisions.h`](../physics/collision/Collisions.h.md) · [`system/Console.h`](../system/Console.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`Terrain.h`](Terrain.h.md)
**Used by** — callers of [`ProceduralRoad.h`](ProceduralRoad.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

A road is a sequence of *blocks*; each block defines an 8-point cross-section; consecutive cross-sections are joined by quads. All quads use one atlas material `road2`; texture fit selects the atlas band.

## State

See [`ProceduralRoad.h`](ProceduralRoad.h.md).

## Cross-section (`computePoints`) — in the block frame (x forward, y up, z left), w = width/2

| Pt | FLAT / LEFT side | BOTH / RIGHT side | BRIDGE | MONORAIL |
|---|---|---|---|---|
| 1 | (0, −bh, bw + w) | (0, bh, bw + w) | (0, bh, bw + w) | same |
| 0 | base of 1 | base of 1 | (0, −0.4, bw + w) | (0, −1.4, bw + w) |
| 2 | (0, −bh/4, bw/3 + w) | (0, bh, w) | (0, bh, w) | same |
| 3 | (0, 0, w) | (0, 0, w) | (0, 0, w) | same |
| 4 | (0, 0, −w) | (0, 0, −w) | (0, 0, −w) | same |
| 5 | (0, −bh/4, −bw/3 − w) (FLAT, RIGHT) | (0, bh, −w) (BOTH, LEFT) | (0, bh, −w) | same |
| 6 | (0, −bh, −bw − w) (FLAT, RIGHT) | (0, bh, −bw − w) (BOTH, LEFT) | (0, bh, −bw − w) | same |
| 7 | base of 6 | base of 6 | (0, −0.4, −bw − w) | (0, −1.4, −bw − w) |

LEFT = sloped left edge + raised right kerb; RIGHT the mirror. `base(p)` = the terrain height at p minus 1 cm (never above p − 1 cm). Points 3–4 are the carriageway.

## `addBlock(pos, rot, type, …)`

```text
IF type AUTOMATIC: width 10, bwidth 1.4, bheight 0.2; measure clearance above terrain at both outer edges (dl, dr):
   both < bh + 0.1 → FLAT; one side high but < 4 m → LEFT/RIGHT; both between → BOTH; otherwise BRIDGE;
   non-flat results use width 10, bwidth 0.4, bheight 0.5
IF first block: cap the start (3 quads, no texture); first = false
ELSE
  IF MONORAIL: pos.y += 2
  carriageway 3–4: ROAD (MONORAIL: CONCRETETOP)
  both FLAT: kerbs 4–5 ROADS3, 2–3 ROADS2, verges 5–6 ROADS4, 1–2 ROADS1
  else: inner walls 4–5, 2–3 CONCRETEWALLI and tops 5–6, 1–2 CONCRETETOP (winding flipped per side so faces point outward)
  BRIDGE or MONORAIL (now or before): outer walls 0–1, 6–7 CONCRETEWALL and underside 7–0 CONCRETEUNDER (flipped); else outer walls BRICKWALL
  pillars for BRIDGE/MONORAIL with pillartype > 0: one per segment under a point between the two sections, shifted toward the higher bank
     (0.8/0.2) when the deck is < 10 m above ground; monorail: centred, only every 5th (global counter), none taller than 20 m, width 0.2;
     length = deck height above ground + 5; half-width = len/30 capped at 5; skipped below 0.2; four CONCRETETOP sides
remember this block as previous
```

`finish` caps the end with three quads, builds the mesh ("RoadSystem-<id>", material `road2`, smooth normals accumulated per vertex), creates the entity under the grouping node, and registers the collision triangles as a collision mesh.

## Quads and collision

`addQuad` silently stops adding once the vertex or triangle budget (50 000) would be exceeded. Winding: (1,2,3)+(1,3,4), or flipped (1,2,4)+(2,3,4). With collision on, each quad becomes two collision triangles with ground model **asphalt** for ROAD/ROADS1–4 fits and **concrete** otherwise.

## Texture fit (atlas v bands; u = distance along the road / 10)

Walls: projected into the frame (along segment, up); BRICKWALL `v = min(0.746 − y·0.25/4.5, 1)`, CONCRETEWALL `min(0.496 − (y − 0.7)·0.25/4.5, 1)`, CONCRETEWALLI `min(0.496 + y·0.25/4.5, 1)`. Horizontal surfaces are projected to the ground plane: CONCRETETOP `v = 0.621 + (z − z₀)·0.25/4.5`; others use fixed bands for the first two / last two corners: ROAD 0.072–0.423, ROADS1 0.001–0.036, ROADS2 0.036–0.072, ROADS3 0.423–0.458, ROADS4 0.458–0.493, CONCRETEUNDER 0.496–0.745. NONE → all (0, 0).
