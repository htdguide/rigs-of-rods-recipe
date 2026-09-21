# source/main/utils/WriteTextToTexture.h

> Rasterise text into an existing texture; save a texture to disk.

**Needs** — [`Application.h`](../Application.h.md) · [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine)
**Used by** — [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md) · [`WriteTextToTexture.cpp`](WriteTextToTexture.cpp.md)
**Tier floor** — T2

## Purpose

Used by legacy dashboards and signs to bake text into textures on the CPU. Implementation: [`WriteTextToTexture.cpp`](WriteTextToTexture.cpp.md).

## State

Stateless.

## `SaveImage(texture, filename)`

**Contract** — reads back mip 0 of the texture and writes it as an image file (format from the extension).

## `WriteToTexture(text, texture, rect, font, colour, size = 15, dpi = 400, justify = 'l', wordwrap = true)`

**Contract** — alpha-blends the glyphs of `text` in `colour` into `rect` of `texture`, clipped to the texture; justify `'l'`, `'c'` or `'r'`; with word wrap, lines break only at spaces/tabs. Text that does not fit vertically is cut off. See the `.cpp` twin.
