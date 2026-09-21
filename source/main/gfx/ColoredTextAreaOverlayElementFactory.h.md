# source/main/gfx/ColoredTextAreaOverlayElementFactory.h

> Registers the coloured text element with the overlay system under type name "ColoredTextArea".

**Needs** — [`Application.h`](../Application.h.md) · [`ColoredTextAreaOverlayElement.h`](ColoredTextAreaOverlayElement.h.md)
**Used by** — [`resources/ContentManager.cpp`](../resources/ContentManager.cpp.md)
**Tier floor** — T2


## Purpose

Overlay scripts refer to the element type by name; this factory creates instances for that name.

## State

Stateless.

## `createOverlayElement(name)` / `getTypeName`

Returns a new [`ColoredTextAreaOverlayElement`](ColoredTextAreaOverlayElement.h.md); type name `"ColoredTextArea"`.
