# source/main/gui/panels/GUI_TextureToolWindow.h

> Debug browser for loaded textures with preview, properties and export.

**Needs** —  · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_TextureToolWindow.cpp`](GUI_TextureToolWindow.cpp.md)
**Tier floor** — T2


## Purpose

Inspect render targets and dynamic textures at runtime. Implementation: [`GUI_TextureToolWindow.cpp`](GUI_TextureToolWindow.cpp.md).

## State

```text
RECORD TextureToolWindow: visible, hovered, "dynamic only" filter (default on), selected texture
CONSTANTS: list pane 200 px, window 600 px
```

## API

`SetVisible`, `IsVisible`, `IsHovered`, `Draw`, `SaveTexture(name, as PNG)`.
