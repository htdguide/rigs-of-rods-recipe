# source/main/gui/panels/GUI_MainSelector.cpp

> Query-driven lists, search syntax, keyboard navigation and what "OK" means per content type.

**Needs** — [`GUI_MainSelector.h`](GUI_MainSelector.h.md) · [`Application.h`](../../Application.h.md) · [`physics/ActorManager.h`](../../physics/ActorManager.h.md) · [`resources/CacheSystem.h`](../../resources/CacheSystem.h.md) · [`resources/ContentManager.h`](../../resources/ContentManager.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [`GUI_LoadingWindow.h`](GUI_LoadingWindow.h.md) · [`utils/InputEngine.h`](../../utils/InputEngine.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`scripting/ScriptEngine.h`](../../scripting/ScriptEngine.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_MainSelector.h`](GUI_MainSelector.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_MainSelector.h`](GUI_MainSelector.h.md).

## State

See header.

## `Show(type, guid, advertised)`

Reset search; restore the last category (default "All") and entry for this type; build lists.

## Lists — `UpdateDisplayLists`

Query the cache with type, category, search method/string and GUID filter (for dashboards, the category the settings screen asked for). Entries: the advertised one first, then results (without duplicating it); select the first. Categories: all with a non-zero result count plus always "Fresh", sorted "All", "Fresh", then by name, titled `(count) name`.

## Search syntax

No colon → full-text (name, file name, description, author name/e-mail). `guid:x`, `author:x`, `wheels:x` (e.g. `4x4`), `file:x` → that field; any other prefix or missing value → no filter. Typing resets the category to "All".

## `Draw`

1/1.4 × 1/1.2 of the screen on first show; captures the keyboard.

- Left/Right cycle categories (unless typing a search); Tab or typing focuses the search box; Up/Down cycle entries with wrap, scrolling to the selection.
- Entry names support colour marks. Click selects (resetting the configuration choice); double-click applies.
- Detail pane: preview image (dashboards at natural size unless too wide; others fitted), name, description, authors `name [role]`, version, wheels `NxM`, mass in tonnes; with *Show details*: load mass, counts (nodes, beams, shocks, hydros, sound sources, commands, rotators, exhausts, flares, flexbodies, props, wings), submeshes, default skin, torque, gear count, RPM range, unique id, GUID, use count, modified/installed dates, vehicle type, and red flags (forwards commands, imports commands, rescuer, custom particles, has fixes, car engine, zip archive, unpacked directory), source path, file name.
- Bottom: configuration combo when the entry has section configs; *OK* (or Enter), *Cancel* (or Escape).

## `Apply`

- Gadget → load its script as GADGET.
- Terrain in the main menu → load terrain.
- Dashboard in the main menu → store as the default truck or boat dashboard (whichever settings asked for).
- In simulation → hand (type, entry, chosen config) to the game context, which spawns or applies it.

Every apply closes the selector.

## `Cancel`

Close; in the main menu also disconnect if a connection was pending and reopen the menu; in simulation notify the game context.
