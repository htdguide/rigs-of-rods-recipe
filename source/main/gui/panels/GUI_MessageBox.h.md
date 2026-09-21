# source/main/gui/panels/GUI_MessageBox.h

> A generic modal message box whose buttons post game messages and/or notify scripts.

**Needs** — [`ForwardDeclarations.h`](../../ForwardDeclarations.h.md) · [`Application.h`](../../Application.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_MessageBox.cpp`](GUI_MessageBox.cpp.md) · [`main.cpp`](../../main.cpp.md) · [`physics/Savegame.cpp`](../../physics/Savegame.cpp.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.cpp`](../../resources/addonpart_fileformat/AddonPartFileFormat.cpp.md)
**Tier floor** — T2


## Purpose

One dialog serves engine prompts (with typed actions) and script message boxes (with numbered buttons). Implementation: [`GUI_MessageBox.cpp`](GUI_MessageBox.cpp.md).

## State

```text
RECORD MessageBoxButton: caption; message type to post (or none) + description + payload; script button number (≥1, or −1)
RECORD MessageBoxConfig: title, text, optional "Always ask" setting, optional external close flag, allow close,
                         content width (default 300), buttons
RECORD MessageBoxDialog: config, close handle (external | own visibility | none), visible
```

## API

`Show(config)`, `Show(title, text, allow close, button1, button2)` (script-style: buttons numbered 1 and 2), `Draw`, `IsVisible`.
