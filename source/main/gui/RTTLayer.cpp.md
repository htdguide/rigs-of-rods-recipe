# source/main/gui/RTTLayer.cpp

> Render-to-texture GUI layer with lazy redraw, pooled because the GUI library cannot destroy layers.

**Needs** — [`RTTLayer.h`](RTTLayer.h.md) · [Seam: Retained layout GUI](../../../SYSTEM-REQUIREMENTS.md#seam-retained-layout-gui)
**Used by** — callers of [`RTTLayer.h`](RTTLayer.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`RTTLayer.h`](RTTLayer.h.md).

## State

See header.

## Layer definition

A layer declared in a layout file may carry properties `TextureSize` and `TextureName`; the texture (RGBA8, render target) is created after reading them. Without a name, one is derived from the layer's identity. Zero size → no texture.

## Rendering

Each GUI render pass: if the layer is out of date (own flag or the library's) or an update is forced, draw all child items into the texture. Otherwise the texture keeps its last contents — static dashboards cost nothing.

## Pool

`CreateOrReuseRttLayer` pops a recycled layer or creates `RttLayer_<n>`. `RecycleRttLayer` pushes it back after its texture is destroyed. The GUI library can create layers at runtime but not remove them, so layers are never destroyed, only reused.
