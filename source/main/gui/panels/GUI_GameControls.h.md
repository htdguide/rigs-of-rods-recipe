# source/main/gui/panels/GUI_GameControls.h

> Key-binding editor: every input event grouped by category, rebinding by pressing keys, saved to the input map file.

**Needs** — [`utils/InputEngine.h`](../../utils/InputEngine.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_GameControls.cpp`](GUI_GameControls.cpp.md) · [`main.cpp`](../../main.cpp.md)
**Tier floor** — T2


## Purpose

Lets players change keyboard bindings without editing `input.map`. Implementation: [`GUI_GameControls.cpp`](GUI_GameControls.cpp.md). Bindings themselves live in [`utils/InputEngine`](../../utils/InputEngine.h.md).

## State

```text
RECORD GameControls
  visible, hovered; column widths (header/body sync); active map file (default map); unsaved changes
  editing: event, trigger, event type, text buffer (1000); interactive capture active; EXPL flag
```

## API

`SetVisible` (closing cancels edits and reopens the main menu), `IsVisible`, `IsHovered`, `IsInteractiveKeyBindingActive`, `Draw`, `DrawEventEditBox`, `ApplyChanges`, `CancelChanges`, `SaveMapFile`, `ReloadMapFile`.
