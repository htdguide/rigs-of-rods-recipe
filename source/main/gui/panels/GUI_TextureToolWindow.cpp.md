# source/main/gui/panels/GUI_TextureToolWindow.cpp

> Lists textures, previews the selected one, and saves it to the user directory.

**Needs** — [`GUI_TextureToolWindow.h`](GUI_TextureToolWindow.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`Application.h`](../../Application.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui) · [`utils/PlatformUtils.h`](../../utils/PlatformUtils.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md)
**Used by** — callers of [`GUI_TextureToolWindow.h`](GUI_TextureToolWindow.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_TextureToolWindow.h`](GUI_TextureToolWindow.h.md).

## State

See header.

## `Draw`

Left: every texture known to the renderer (static ones hidden when "dynamic only"). Right: preview fitted to the pane width and at most half the window height; resolution, byte size, pixel format, faces, FSAA, mipmaps, type, usage flags, depth; buttons "Save as PNG" and "Save Raw". Keyboard goes to the GUI while hovered.

## `SaveTexture(name, png)`

Read the texture back into an image and write it to the user directory as the name with `/` → `_` (plus `.png` when requested; otherwise the image codec is chosen by the name's own extension). Result or error goes to the console.
