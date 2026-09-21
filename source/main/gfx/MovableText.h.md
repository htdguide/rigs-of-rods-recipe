# source/main/gfx/MovableText.h

> A camera-facing 3D text label (billboarded glyph quads) for the scene.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`gameplay/Character.cpp`](../gameplay/Character.cpp.md) · [`GfxActor.cpp`](GfxActor.cpp.md) · [`MovableText.cpp`](MovableText.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/collision/Collisions.cpp`](../physics/collision/Collisions.cpp.md)
**Tier floor** — T2


## Purpose

A renderable text object (classic community snippet) used for in-world labels in debug and diagnostic features. Implementation: [`MovableText.cpp`](MovableText.cpp.md).

## State

```text
RECORD MovableText
  name, caption, font ("highcontrast_black"), char height = 1, space width, colour = black,
  alignment: horizontal LEFT | CENTER, vertical BELOW | ABOVE; additional height; on-top (ignore depth)
  geometry (quads), bounding box, radius, material (font material clone), dirty flags
```

## API

Setters/getters for font, caption, colour, char height, space width, alignment, additional height, show-on-top; renderer callbacks (world transform, render op, queue update).
