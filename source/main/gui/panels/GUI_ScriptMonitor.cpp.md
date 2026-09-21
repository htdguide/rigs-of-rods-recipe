# source/main/gui/panels/GUI_ScriptMonitor.cpp

> Draws the running/recent script table and turns clicks into load/unload messages.

**Needs** — [`GUI_ScriptMonitor.h`](GUI_ScriptMonitor.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`resources/ContentManager.h`](../../resources/ContentManager.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`scripting/ScriptEngine.h`](../../scripting/ScriptEngine.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_ScriptMonitor.h`](GUI_ScriptMonitor.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_ScriptMonitor.h`](GUI_ScriptMonitor.h.md).

## State

See header.

## `Draw`

Columns ID (25 px), file name (200), options (200).

**Active** — every script unit: id; the gadget file (for gadgets) or script name; then by category:
- ACTOR: `(actor) [index] 'name'`.
- TERRAIN: `(terrain)`.
- CUSTOM / GADGET: *Reload* (post unload, then chain a load of the same file and category so it runs after the unload), *Stop* (post unload), *Autoload* checkbox that adds/removes the file in the `app_custom_scripts` list.

**Recent** — entries of `app_recent_scripts` not currently running: *Load* (as CUSTOM) and *Remove* (from the list).

Section headers are drawn as a separator with a small labelled tab.
