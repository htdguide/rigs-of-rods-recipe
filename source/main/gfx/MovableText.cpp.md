# source/main/gfx/MovableText.cpp

> Glyph quad generation and billboard transform for 3D text.

**Needs** — [`MovableText.h`](MovableText.h.md)
**Used by** — callers of [`MovableText.h`](MovableText.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`MovableText.h`](MovableText.h.md).

## State

See header.

## Behaviour

- Geometry: one quad per glyph from the font's texture coordinates and aspect ratio, spaces advance by the space width, newlines start a new line; horizontal centring shifts by half the line length; vertical alignment places the text above or below the origin (+ additional height); bounding box and radius from all vertices.
- Colour lives in a separate vertex stream so colour changes do not rebuild geometry.
- Transform: the parent node's position with the camera's orientation (always facing the camera); scale from the parent.
- On-top mode disables depth check and enables depth write on the private material.
