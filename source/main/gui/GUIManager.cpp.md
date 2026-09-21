# source/main/gui/GUIManager.cpp

> GUI startup, theme, per-state draw lists, cursor and keyboard-capture policy, and global UI hotkeys.

**Needs** — [`GUIManager.h`](GUIManager.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`AppContext.h`](../AppContext.h.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`gfx/camera/CameraManager.h`](../gfx/camera/CameraManager.h.md) · [`resources/ContentManager.h`](../resources/ContentManager.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gfx/GfxActor.h`](../gfx/GfxActor.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`GUIUtils.h`](GUIUtils.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [Seam: Immediate-mode GUI](../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui) · [`OverlayWrapper.h`](OverlayWrapper.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md) · [`RTTLayer.h`](RTTLayer.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [Seam: Retained layout GUI](../../../SYSTEM-REQUIREMENTS.md#seam-retained-layout-gui)
**Used by** — callers of [`GUIManager.h`](GUIManager.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUIManager.h`](GUIManager.h.md).

## State

See header.

## Startup

Initialise the retained GUI on the render window and main scene (log file only if the diagnostic setting asks), register the render-to-texture layer type, load core skin and English font resources, hide its cursor (the immediate GUI draws the only cursor), forward translation tags on Windows. Then initialise the immediate GUI with the game's dark-blue theme (window bg .06 grey at 90%, accents (.13,.40,.60) hover / (.18,.53,.79) active, padding 10, frame rounding 2, window rounding 4, centred titles, item spacing 5, grab rounding 3, no window border) and hook its renderer into the scene's render queue. The chat box uses the theme's semi-transparent text background.

## Draw lists

| When | Panels drawn (each only if visible) |
|---|---|
| `DrawCommonGui` (both states) | multiplayer client list (if connected, GUI shown, map closed), vehicle/terrain selector, console, controls editor, repository browser |
| `DrawMainMenuGui` | common + multiplayer selector, main menu, settings, message box, loading window, about |
| `DrawSimulationGui` (sim thread synced) | top menu bar (+ main menu when paused), node/beam utils, collisions debug, message box, flexbody debug |
| `DrawSimGuiBuffered` (from sim buffers) | common + vehicle info panel (unless paused menu), chat box (unless console open or GUI hidden), loading window, friction settings, perf stats, texture tool, survey map; direction arrow update |

The split matters: panels that edit live simulation objects draw only while the physics thread is synced; the rest read snapshots.

## Keyboard capture

Panels call `RequestGuiCaptureKeyboard(true)` while a text field has focus; requests OR together during a frame and take effect after it (`ApplyGuiCaptureKeyboard`), so the next frame's game input ignores the keyboard.

## `AreStaticMenusAllowed`

False in free camera or while the mouse is over the console, controls, friction, texture tool, node/beam, collisions, selector, survey map or flexbody windows — prevents the top menu bar popping over them.

## Mouse cursor

Moving the mouse (`WakeUpGUI`) shows the cursor unless suppressed; after 5 s without movement it hides.

## `ApplyUiPreset`

Writes each preset table row's value for the selected `ui_preset` into its setting.

## `SetGuiHidden(h)`

Stores the setting, shows/hides the overlay dashboards accordingly, and on hide also closes perf stats and chat.

## Menu wallpaper

Random `.jpg` from the "Wallpapers" group (else `.png`), shown as a full-screen overlay panel at z-order 0.

**Notes** — the random pick re-rolls a random number of times (0–10) instead of picking once; equivalent to one uniform pick. With no wallpaper files at all the original indexes an empty list.

## `UpdateInputEvents` — global hotkeys

Console toggle (always). In simulation: hide GUI; chat (0.5 s debounce, multiplayer only); vehicle info stats tab / commands tab (toggle, only with a player vehicle); dashboard overlay toggle; perf stats; survey map cycle/toggle (not in free camera).
