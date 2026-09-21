# source/main/gui/panels/GUI_TopMenubar.cpp

> Menu contents, hover policy, the state box, AI preset loading, and the tune-up editor.

**Needs** — [`GUI_TopMenubar.h`](GUI_TopMenubar.h.md) · [`Application.h`](../../Application.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`physics/ActorManager.h`](../../physics/ActorManager.h.md) · [`gfx/camera/CameraManager.h`](../../gfx/camera/CameraManager.h.md) · [`DashBoardManager.h`](../DashBoardManager.h.md) · [`physics/flex/FlexBody.h`](../../physics/flex/FlexBody.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [`GUI_MainSelector.h`](GUI_MainSelector.h.md) · [`utils/InputEngine.h`](../../utils/InputEngine.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`network/Network.h`](../../network/Network.h.md) · [`utils/PlatformUtils.h`](../../utils/PlatformUtils.h.md) · [`gameplay/Replay.h`](../../gameplay/Replay.h.md) · [`gfx/SkyManager.h`](../../gfx/SkyManager.h.md) · [`terrain/Terrain.h`](../../terrain/Terrain.h.md) · [`resources/terrn2_fileformat/Terrn2FileFormat.h`](../../resources/terrn2_fileformat/Terrn2FileFormat.h.md) · [`resources/tuneup_fileformat/TuneupFileFormat.h`](../../resources/tuneup_fileformat/TuneupFileFormat.h.md) · [`gfx/GfxWater.h`](../../gfx/GfxWater.h.md) · [`scripting/ScriptEngine.h`](../../scripting/ScriptEngine.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`resources/ContentManager.h`](../../resources/ContentManager.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_TopMenubar.h`](GUI_TopMenubar.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_TopMenubar.h`](GUI_TopMenubar.h.md). Drawn while the simulation is synced (it edits live state).

## State

See header.

## Showing the bar — `ShouldDisplay`

Hidden when static menus aren't allowed or the right mouse button is down. Otherwise shown while the mouse is in the top 50 px band, inside the open menu's box (+25×10), or inside the state box; hovering the state box closes any open menu. The AI menu stays open while a vehicle is being picked for it. Buttons: Simulation, [Tuning], [Vehicle AI], Vehicles (count), Saves, Settings, Tools — centred; hovering a button opens its menu below it. AI and Tuning only offline (Tuning also needs the setting).

## Simulation menu

Get new vehicle (selector, all vehicle types); with a player vehicle: show description (vehicle panel, commands tab), and for local vehicles reload / remove; otherwise for the last spawned vehicle: activate, reload, remove. With local vehicles: remove all (with confirm; skips terrain-preloaded, hidden-from-list and remote), activate all, "activated vehicles never sleep", send all to sleep. Offline: reload terrain (unload, reload its bundle, load again) and, in editor mode on an unpacked terrain, save terrain changes. Back to menu (disconnecting if online); Exit.

## Vehicles menu

Offline: every listed vehicle — `X` removes, the name button seats the player; name coloured by: current, linked to current, simulated, sleeping. Online: per user `name: count (rank, Ver, Lang)`, then their vehicles — remote ones with hide/unhide buttons, own ones with `X`; clicking a name seats the player.

## Saves menu

Quicksave (numpad /) and, when a quicksave file exists, Quickload (numpad *); slots 1–5 `quicksave-<n>.sav` shown by scene name — save (Ctrl+Alt+n) and load (Alt+n).

## Settings menu

Volume; reflections update rate; FPS limit; slow motion 0.01–1 and time lapse 1–10 (one simulation speed); camera FOV and height (static camera), or interior/exterior FOV (+ tracking for fixed camera); time of day ±0.5 around the value at opening, day cycle and speed 10–2000 (Caelum sky); waves height 0–4 (offline, basic water with waves); keyboard steering speed coupling; online: collisions and hide labels.

## Tools menu

Friction settings, console, texture tool, collisions debug, node/beam utility and flexbody debug (with a vehicle), browse gadgets, browse repository; diagnostic logging toggles (mass recalculation, beam break, beam deform, triggers, video camera markers); polygon mode 1 solid / 2 wireframe / 3 points.

## Vehicle AI menu

Count, following distance, position scheme, repeat count, mode, speed (km/h or knots), altitude (ft), vehicle choice(s) via the selector. Mode rules: *Drag race* forces 2 vehicles, speed 1000, parallel, once; *Crash* forces 2 vehicles, speed 100, opposite, once (previous values restored when leaving either); *Chase* follows the player: the only waypoint is 20 m +X of the player. Mode is locked while AI vehicles exist.

- *Start* (with waypoints or chase) loads `AI.as` as a custom script; without waypoints, a hint points to presets, recording or the map.
- *Stop* deletes all AI vehicles (and chase waypoints).
- *Record* clears and starts recording waypoints (the recording itself is done by the game loop); *Recording* stops.
- *Presets*: entries whose `terrain` equals the loaded terrain file become buttons that replace the waypoints; waypoint arrays `[x, y, z]` or `[x, y, z, speed]` (speed < 5 ignored). External presets are fetched on first open (spinner, error + Retry); none for this terrain → list of supported terrains from entries with a `terrains` array.
- *Waypoints*: *Export* logs a ready-to-paste JSON preset (`terrain`, `preset`, `waypoints`) to the log file; per waypoint: teleport button and speed (−1 default, ≥5 override).

### AI preset sources

Bundled: every JSON file listed in the terrain's `[AI Presets]` (root must be an array). External, on a worker thread: `savegames/waypoints.json` if present, else `https://raw.githubusercontent.com/RigsOfRods-Community/ai-waypoints/main/waypoints.json`; result posted as a success/failure message. The merged list = bundled then external.

## Tuning menu

Needs a player vehicle. Rebuilt when the player vehicle changes (`RefreshTuningMenu`): eligible addon parts = cache entries of type addon part matching the vehicle GUID and file name, plus any already in use; saved tune-ups for this vehicle (user category); pairwise addon-part conflicts; which parts conflict with used ones.

- Saved tune-ups: click to load; *Delete* (hold 1.5 s).
- *Save as…* → name, *Save* (create tune-up project; *Overwrite* option), *Cancel*; *Reset* (hold 1.5 s).
- Addon parts: checkbox per part (blocked, drawn with an ✕, when it conflicts with a used part; a red frame marks parts conflicting with the hovered one); *Reload* per part (reload its bundle, then respawn the vehicle in place with the same config, skin, tune-up and debug view); *Browse all parts*.
- Props, flexbodies, flares (named by type letter, custom number or dashboard link), exhausts, managed materials: enable checkbox (unchecking force-removes; *Reset* clears a forced removal) and *Protected* (addon parts can't touch it).
- Wheels: L/R side radio (forces a side; *Reset*), rim mesh name, *Protected*.
- Video cameras: material name; mirrors get a *Flipped* toggle (switches mirror ↔ no-flip variants); *Reset*.

Every change posts a modify-project request; the vehicle is updated by the handler.

## State box (centred below the bar; always drawn, even when the bar is hidden)

First match wins: all physics paused (orange, resume key) · player vehicle physics paused (green) · replay (progress bar frame/total, elapsed time) · live repair (green; optional control legend: move forward/back/left/right/up/down, rotate, Alt slow / Shift fast / Ctrl 10× step, reset-mode switch) · quick repair (orange; timer bar until live repair) · race (checkpoint text, distance in metres, time `MM.SS.hh` red/green/blue by delta, best time) · terrain editor (read-only terrain → *Import as editable project* runs `terrain_project_importer.as` once; unpacked → "Use Simulation menu to save").
