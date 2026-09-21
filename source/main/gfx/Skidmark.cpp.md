# source/main/gfx/Skidmark.cpp

> skidmarks.cfg parsing and ribbon growth with segment rollover.

**Needs** — [`Skidmark.h`](Skidmark.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`Application.h`](../Application.h.md) · [`physics/SimData.h`](../physics/SimData.h.md) · [`resources/ContentManager.h`](../resources/ContentManager.h.md) · [`GfxScene.h`](GfxScene.h.md) · [`utils/Utils.h`](../utils/Utils.h.md)
**Used by** — callers of [`Skidmark.h`](Skidmark.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`Skidmark.h`](Skidmark.h.md).

## State

See header.

## Config file

`skidmarks.cfg` (config group): `;` comments; a single word starts a model; lines `ground, texture, slip_from, slip_to` add a rule. A texture applies when `slip_from ≤ slip < slip_to` for that ground. Load errors clear all rules (no marks).

## `update(contact, index, slip, ground)`

```text
texture = rule for model "default"; none → RETURN
axis = axle vector (flipped for odd contact indices); centre point = contact + axis/2
gap limit = max_gap × wheel speed (when faster than 1)
IF no segment: start one at the contact
ELSE distance from the last centre; < 0.25 m → RETURN
  texture differs from the segment's first texture, or the segment is full:
    far (> gap limit) → start a new segment at the contact; else continue: new segment seeded with the last two points
  ELSE IF distance > max gap: new segment
append two points: contact − 0.2·axis and contact + 1.2·axis (face size = distance)
rebuild the strip: points past the write position collapse onto the last valid point; UVs cycle through the 4 corners,
  u scaled by face size / 0.25; bounding box from all points
```

Segments beyond 20 are discarded oldest-first (material and mesh destroyed). Materials are transparent, unlit, no depth write, depth-biased, double-sided; marks render up to 800 m.
