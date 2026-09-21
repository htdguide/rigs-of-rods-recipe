# source/main/gui/panels/GUI_GameControls.cpp

> Draws the binding table, captures key combos, and rewrites bindings as input-map lines.

**Needs** — [`GUI_GameControls.h`](GUI_GameControls.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`Application.h`](../../Application.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui) · [`GUI_LoadingWindow.h`](GUI_LoadingWindow.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`utils/InputEngine.h`](../../utils/InputEngine.h.md)
**Used by** — callers of [`GUI_GameControls.h`](GUI_GameControls.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_GameControls.h`](GUI_GameControls.h.md).

## State

See header.

## Table

800×600 window with *Save changes* / *Reset changes* when dirty, and tabs Airplane, Boat, Camera, Sky, Character, Commands, Common, Grass, Map (SURVEY_MAP), Menu, Truck, Road editor — each lists events whose name starts with that prefix. Only keyboard triggers from the user map or built-in defaults are shown; events with none are omitted. A row: event name, one button per binding (numbered when several), description.

## Rebinding

Clicking a binding starts interactive capture for that event and trigger, keeping its EXPL flag. A centred "Press a new key" window shows the event, its description, the keys currently held and an EXPL checkbox (tooltip: with EXPL only exact combos fire; without, partial matches fire too — e.g. Ctrl+F1 triggers both `F1` and `CTRL+F1` bindings but not `EXPL+F1`). As soon as a non-modifier key is down: binding = `EXPL+<combo>` or `<combo>`, apply, and flush the pressed keys so they don't reach gameplay. Closing the window cancels.

## `ApplyChanges`

Empty buffer → cancel. Otherwise remove the old trigger and feed the input engine one map-file line: `<EVENT> <Type> <binding>` for keyboard, `<EVENT> <Type> 0 <binding>` for joystick types; mark unsaved.

## Save / reset

Save writes the active map file; reset clears that file's bindings and reloads it.

## `DrawEventEditBox` (manual editor)

Type combo (mouse and relative-axis types excluded), text field (no blanks for keys/buttons; upper-case except POV), OK / Cancel / Delete.
