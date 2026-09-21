# source/main/gfx/ColoredTextAreaOverlayElement.h

> An overlay text element that understands ^0–^9 colour codes.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`ColoredTextAreaOverlayElement.cpp`](ColoredTextAreaOverlayElement.cpp.md) · [`ColoredTextAreaOverlayElementFactory.h`](ColoredTextAreaOverlayElementFactory.h.md)
**Tier floor** — T2


## Purpose

Legacy overlay text (e.g. HUD/race messages) with inline colour codes. Implementation: [`ColoredTextAreaOverlayElement.cpp`](ColoredTextAreaOverlayElement.cpp.md); registered with the overlay system through [`ColoredTextAreaOverlayElementFactory.h`](ColoredTextAreaOverlayElementFactory.h.md).

## State

```text
RECORD ColoredTextAreaOverlayElement = { per-character colour ids; value_top = 1.0; value_bottom = 0.8 }
```

## API

`setCaption(text)`, `StripColors(text)`, `GetColor(id, value)`, `setValueTop/Bottom`, `updateColours`.
