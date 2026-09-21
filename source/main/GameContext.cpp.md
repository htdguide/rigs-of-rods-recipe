# source/main/GameContext.cpp

> Message queue mechanics, spawn/modify/delete rules, player seating, loader flow, and the per-frame input mapping for every vehicle type.

**Needs** — [`GameContext.h`](GameContext.h.md) · [`AppContext.h`](AppContext.h.md) · [`physics/Actor.h`](physics/Actor.h.md) · [`physics/air/AeroEngine.h`](physics/air/AeroEngine.h.md) · [`resources/CacheSystem.h`](resources/CacheSystem.h.md) · [`physics/collision/Collisions.h`](physics/collision/Collisions.h.md) · [`system/Console.h`](system/Console.h.md) · [`gui/DashBoardManager.h`](gui/DashBoardManager.h.md) · [`gameplay/Engine.h`](gameplay/Engine.h.md) · [`gfx/GfxScene.h`](gfx/GfxScene.h.md) · [`gui/GUIManager.h`](gui/GUIManager.h.md) · [`gui/panels/GUI_FrictionSettings.h`](gui/panels/GUI_FrictionSettings.h.md) · [`gui/panels/GUI_MainSelector.h`](gui/panels/GUI_MainSelector.h.md) · [`gui/panels/GUI_TopMenubar.h`](gui/panels/GUI_TopMenubar.h.md) · [`utils/InputEngine.h`](utils/InputEngine.h.md) · [`gui/OverlayWrapper.h`](gui/OverlayWrapper.h.md) · [`gameplay/Replay.h`](gameplay/Replay.h.md) · [`physics/water/ScrewProp.h`](physics/water/ScrewProp.h.md) · [`scripting/ScriptEngine.h`](scripting/ScriptEngine.h.md) · [`gfx/SkyManager.h`](gfx/SkyManager.h.md) · [`gfx/SkyXManager.h`](gfx/SkyXManager.h.md) · [`audio/SoundScriptManager.h`](audio/SoundScriptManager.h.md) · [`terrain/Terrain.h`](terrain/Terrain.h.md) · [`resources/terrn2_fileformat/Terrn2FileFormat.h`](resources/terrn2_fileformat/Terrn2FileFormat.h.md) · [`resources/tuneup_fileformat/TuneupFileFormat.h`](resources/tuneup_fileformat/TuneupFileFormat.h.md) · [`utils/Utils.h`](utils/Utils.h.md) · [`gameplay/VehicleAI.h`](gameplay/VehicleAI.h.md)
**Used by** — callers of [`GameContext.h`](GameContext.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GameContext.h`](GameContext.h.md).

## State

See header.

## Message queue

All four operations take the queue lock. Push appends and remembers the new message as the chain end. Chain appends to the chain end's chain (and moves the chain end into it); with no chain end it behaves like push. Pop clears the chain end if it is the popped message. (The main loop re-posts a processed message's chain.)

## `LoadTerrain(name)`

Find the terrain in the content cache (partial file-name match; missing → message box), load its bundle, parse the `.terrn2` ([`resources/terrn2_fileformat`](resources/terrn2_fileformat/README.md)), load its declared asset packs, build and initialise the [`Terrain`](terrain/Terrain.h.md) (failure → release, false). Then render the environment map once from the terrain centre and let the friction editor list the ground models.

## `SpawnActor(request)` — returns the actor or none

```text
USER origin: remember cache entry, skin, tune-up, config for "respawn last";
  without a spawn box: place at the player vehicle (same heading − 270°, same height above ground) or at the character (180° − heading)
resolve the cache entry by file name if missing; fetch the parsed definition (errors reported → none)
skin: load; drop if its .skin failed. Tune-ups only offline with tuning enabled, else none
local spawn while online: stamp owner name and colour for labels
create the actor
definition says slide nodes connect instantly → lock slide nodes now
by origin:
  USER: last spawned = it; seat player if drivable; outside a spawn box push it out of collisions (radius 50; also terrain if on foot)
  CONFIG_FILE (preset): seat if drivable, has nodes and "enter preselected" is set
  TERRN_DEF: terrain "machine" flag → drivable type MACHINE
  AI: type AI, simulated, gearbox automatic in Drive
  NETWORK: remember source/stream ids; apply peer options (mute, hide) via messages
  SAVEGAME: queue "restore saved state"
  otherwise: seat if drivable and the request asks to enter
seating is a SEAT_PLAYER message followed (chained) by a tuning-menu refresh
```

## `ModifyActor(request)`

SOFT_RESET, RESET_ON_SPOT (keep position), RESET_ON_INIT_POS, RESTORE_SAVED, WAKE_UP (sleeping → simulated), SOFT_RESPAWN (to a given pose), REFRESH_VISUALS (rebuild sim buffer and force a graphics update). RELOAD: refuse if the cache entry was deleted; otherwise reload the bundle, then (chained) spawn a copy at the same ground position and heading with the same config, skin, working tune-up and debug view — the old actor is deleted by whoever requested the reload.

## `DeleteActor(actor)`

If driven, get out first and put the character at the vehicle's centre. Forget it as previous/last spawned. Release ties and hooks of other local actors linked to it. Remove its graphics; online, undo remote characters seated in it. Fire `GENERIC_DELETED_TRUCK`. Delete.

## `ChangePlayerActor(actor)`

Switch 3D dashboards (old off, new on); hide the old HUD overlay and stop its air and pump sounds.

- **Getting out** — mirror cameras go offline; leave the interior; place the character beside the vehicle: with a cinecam, try 2 m left or right of it along the roll axis, choosing the side whose ground is closer (right preferred unless 20 % worse); drop to the surface; face the vehicle's heading − 90°. Force feedback off. Fire `TRUCK_EXIT`.
- **Getting in** — show HUD overlay (unless GUI hidden); mirror cameras online; force feedback only for trucks; seat the character; analyse flexbodies for the debug panel. Fire `TRUCK_ENTER`.

Notify the camera and update sleeping states.

## Loader flow (vehicle selector)

- `ShowLoaderGUI(type, instance, box)` — for a terrain spawn box: offline, refuse ("Please clear the place first") if any node is inside it; remember the box's position, direction and the box itself; open the selector for that type.
- `OnLoaderGuiApply(type, entry, config)` — addon part: add to the player vehicle's tune-up. Skin: use it (not the dummy default) and spawn. Vehicle types: remember entry and config (origin USER); if the vehicle has a GUID, query its skins — a declared default skin is used automatically when it is the only one (missing default → warning); otherwise, if skins exist, reopen the selector for skins with a "Default skin" entry advertised on top; else spawn. When picking for the AI panel, store name/config/skin there instead of spawning. The loader context is reset after spawning.
- `OnLoaderGuiCancel` — reset context; return to the AI menu if it was picking.

## Characters

`CreatePlayerCharacter` — create the local character at the terrain spawn point; rotation from the terrain definition, else 180° when the terrain has no predefined vehicles; command-line then diagnostic presets override position/rotation (each consumed once); drop onto the surface from 1.8 m above; place the camera and run 100 camera updates of 20 ms to settle it.

## `TeleportPlayer(x, z)`

On foot: move the character to the surface. In a vehicle: fire `TRUCK_TELEPORT`; move the vehicle and every linked actor by the same translation, raised so the lowest node keeps its original height above ground (never below 0).

## Input — global (every frame)

Escape: in the main menu close the top-most window (about, selector, settings, controls, multiplayer, repository) or quit; in simulation close the selector or controls, else pause + open menu (offline pause only) or resume. Screenshot (0.25 s debounce); fullscreen toggle (2 s); polygon mode; log current position/rotation; toggle soft/hard reset mode.

## Input — simulation

New vehicle selector; enter/exit (0.5 s): on foot, enter the nearest drivable vehicle with a cinecam within 20 m of head height; in a vehicle, leave only when nearly stopped (|v| < 1 m/s) or it is remote/AI. Next/previous vehicle; respawn last; terrain editor (on foot).

**Command import** — on foot, the nearest actor that imports commands and whose centre is within its camera radius receives the command keys 1–N from the player's input (waking it if a value changes); switching or leaving resets its keys to 0.

**Waypoint recording** (AI menu) — every second: in a vehicle, add its position if ≥ 5 m from the last (speed = wheel speed km/h, below 5 → default); on foot, add the position if changed.

## Input — sky

Caelum: time factor 1000 / 10000 / −1000 / −10000 while the time keys are held, else the cycle speed (or 1); a change is announced with the new time. SkyX: multiplier 1, 2, −1, −2, else 0.01.

## Input — any player vehicle

Reload (0.5 s); remove; low beams (with side lights); light cycle side → head → high → off (skipping absent kinds); high beams; fog; beacons; blinkers left/right/hazard; custom lights 1–10; remove; rope lock; lock (hook + slide-node requests); release auto-locks; secure load (ties); custom particles; debug view toggle/cycle (applied to linked actors too); rescue (offline, not aircraft: seat in the vehicle flagged rescuer, else notice); trailer parking brake (trucks) or parking brake (loads); video cameras (0.5 s); while holding enter/exit, brake 0.66; physics pause (with linked actors); reset to initial position; command keys from input (plus the one held on the vehicle panel); forward-commands and import-commands toggles with notices; replay controls.

## Input — aircraft

Skipped while resetting or paused. Autopilot disconnect request honoured. Aileron, elevator (down − up) and rudder (left − right) move toward the input at 4/s (`smoothValue`: clamp target to ±1, step by rate). Steering is speed-coupled unless both steer inputs are analog. Brake 0.66 × input unless parked; parking brake; reverse and start toggles for all engines; flaps and air brakes 0–5 (none/full/less/more). Throttle: the throttle key sets full; a throttle axis sets it; up/down ±0.05 (0.1 s); none/full. The autopilot finally rewrites each engine's throttle.

## Input — boat

Throttle axis mapped 0…1 → −1…1 and applied negated; up/down ±0.05; keyboard rudder integrates (left − right)·dt when not debounced; rudder axes set it directly; centre; reverse.

## Input — truck

Skipped while resetting, paused, or AI-driven. Mirror angles ±0.001 per press. Steering = −max(left digital, left analog) + max(right digital, right analog), clamped; speed coupling when digital dominates. Engine inputs. Brake sound above 1/6 brake. Differential and transfer-case toggles with on-screen notices. Horn: police siren toggles; otherwise held (key or panel button). Parking brake (unless the trailer brake key is held), ABS, TC, cruise control; tyre pressure keys.
