# source/main/gfx/ColoredTextAreaOverlayElement.cpp

> Colour-code parsing and per-glyph vertical gradient colouring.

**Needs** — [`ColoredTextAreaOverlayElement.h`](ColoredTextAreaOverlayElement.h.md)
**Used by** — callers of [`ColoredTextAreaOverlayElement.h`](ColoredTextAreaOverlayElement.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`ColoredTextAreaOverlayElement.h`](ColoredTextAreaOverlayElement.h.md).

## State

See header.

## Behaviour

- Palette (scaled by a brightness value): 0 black, 1 red, 2 green, 3 yellow, 4 blue, 5 cyan, 6 magenta, 7 white, 8 grey (0.9), 9 dark blue (0.5, 0.5, 0.9·v); default 9.
- `setCaption` — `^d` sets the colour for all following visible characters (spaces and newlines do not consume glyph slots); the displayed text has the codes stripped.
- `updateColours` — each glyph's six vertices get the top colour (brightness `value_top`) on the upper vertices and the bottom colour (`value_bottom`) on the lower ones: a subtle vertical gradient.

**Notes** — `setValueBottom` writes the top value and `setValueTop` the bottom value (swapped in the original). Callers compensate implicitly; keep or fix together.
