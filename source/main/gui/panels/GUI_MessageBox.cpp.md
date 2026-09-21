# source/main/gui/panels/GUI_MessageBox.cpp

> Draws the message box and dispatches button actions.

**Needs** — [`GUI_MessageBox.h`](GUI_MessageBox.h.md) · [`Application.h`](../../Application.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`scripting/ScriptEngine.h`](../../scripting/ScriptEngine.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_MessageBox.h`](GUI_MessageBox.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_MessageBox.h`](GUI_MessageBox.h.md).

## State

See header.

## `Show`

Store the config; the close button uses the external flag if given, else the dialog's own visibility if closing is allowed, else none. Log title, text and buttons.

## `Draw`

Centred on first appearance, fixed content width, wrapped text, one button per entry, optional "Always ask" checkbox bound to its setting. Keyboard goes to the GUI while hovered.

Button click: fire script event `GENERIC_MESSAGEBOX_CLICK(button number)` if the number is ≥ 1; post its message (description, payload) if any; close. Closing with the window's close button fires `GENERIC_MESSAGEBOX_CLICK(0)`.
