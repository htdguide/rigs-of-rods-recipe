# source/main/physics/Savegame.cpp

> The savegame format (JSON, format 3): what a scene snapshot contains and how it is restored onto existing or respawned actors.

**Needs** — [`Application.h`](../Application.h.md) · [`air/AeroEngine.h`](air/AeroEngine.h.md) · [`Actor.h`](Actor.h.md) · [`water/Buoyance.h`](water/Buoyance.h.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`resources/ContentManager.h`](../resources/ContentManager.h.md) · [`system/Console.h`](../system/Console.h.md) · [`gameplay/Engine.h`](../gameplay/Engine.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`gui/panels/GUI_MessageBox.h`](../gui/panels/GUI_MessageBox.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md) · [`water/ScrewProp.h`](water/ScrewProp.h.md) · [`gfx/Skidmark.h`](../gfx/Skidmark.h.md) · [`gfx/SkyManager.h`](../gfx/SkyManager.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`resources/tuneup_fileformat/TuneupFileFormat.h`](../resources/tuneup_fileformat/TuneupFileFormat.h.md) · [`utils/Utils.h`](../utils/Utils.h.md)
**Used by** — callers of [`GameContext.h`](../GameContext.h.md) and [`ActorManager.h`](ActorManager.h.md) — it implements their savegame methods
**Tier floor** — T2

## Purpose

Implements `GameContext` quicksave plumbing and `ActorManager.LoadScene/SaveScene/RestoreSavedState`. It lives in the physics chapter because almost all of its content is actor state. Savegames are files in the savegames resource group; users keep them across versions, so the format is a compatibility surface (see [SYSTEM-REQUIREMENTS](../../../SYSTEM-REQUIREMENTS.md)).

## State

Stateless. Format version constant: **3**.

## File format

```text
{
  "format_version": 3,
  "scene_name":   "<terrain pretty name> [<local actor count>]",
  "terrain_name": "<terrain file name>",
  "daytime":      number                        # only with the Caelum sky
  "forced_awake": bool, "physics_paused": bool,
  "player_position": [x, y, z], "player_rotation": radians,
  "actors": [ ACTOR, ... ]                      # local actors only, in list order
}
ACTOR = {
  "filename": "<bundle file name>:<truck file>",       # bundle-qualified
  "position": node0 [x,y,z], "rotation": heading rad, "min_height": lowest node y,
  "spawn_rotation", "preloaded_with_terrain", "sim_state" (int ActorState), "physics_paused",
  "player_actor", "prev_player_actor", "skin"?: name, "tuneup_document"?: full .tuneup text,
  "section_config",
  engine only: "engine_gear", "engine_rpm", "engine_auto_mode", "engine_auto_select",
               "engine_is_running", "engine_has_contact", "engine_wheel_spin", "alb_mode", "tc_mode",
               "cc_mode", "cc_target_rpm", "cc_target_speed",
  "hydro_dir_state", "hydro_aileron_state", "hydro_rudder_state", "hydro_elevator_state",
  "parking_brake", "trailer_parking_brake", "avg_wheel_speed", "wheel_speed", "wheel_spin", "custom_particles",
  "lights" (int, headlights), "blink_type", "beacon_light", "high_beams_on", "fog_lights_on",
  "custom_lights": [10 × bool], "buoyance_sink"?,
  "aeroengines": [{rpm, reverse, ignition, throttle}], "screwprops": [{rudder, throttle}],
  "rotators": [angle], "wheels": [detached], "wheel_diffs": [active type], "axle_diffs": [active type],
  "transfercase"?: {"4WD": bool, "GearRatio": first ratio},
  "commands": [ [commandValue, triggerInputValue, [[auto_moving_mode, pressed_center_mode] per beam]] × 84 ],   # keys 0..83
  "hooks": [{locked, lock_node, locked_actor}], "ropes": [{locked, locked_ropable, locked_actor}],
  "ties": [{tied, tying, locked_ropable, locked_actor}], "ropables": [{attached_ties, attached_ropes}],
  "slidenodes_locked",
  "nodes": [[abs x,y,z, vel x,y,z, initial x,y,z]],
  "beams": [[maxposstress, maxnegstress, minmaxposnegstress, strength, L, broken, disabled, inter_actor, locked_actor]]
}
```

`locked_actor` fields are indices into the saved `actors` array (−1 = none). Command key slots are 1-based in the simulation; the file stores slots 0..83 and so omits key 84 — kept "to preserve compatibility".

**Notes** — the file format 2 wrote the beacon flag as `pp_beacon_light` (a rename accident); the loader accepts both. `high_beams_on`/`fog_lights_on` appeared in format 3 and default to off.

## `SaveScene(filename)`

**Contract** — refuses in multiplayer for `autosave.sav` and when more than 3 local actors exist ("Too many vehicles"). Builds the document above from the local actors; the tuneup is exported into a zeroed buffer and embedded as a string. Writes via the content manager; failure → "Error while saving scene". Success → "Scene saved" (silent for autosave).

## `LoadScene(filename)`

**Contract** — returns false with a console error when the file is missing/invalid or `format_version ≠ 3`. In multiplayer: autosave is ignored, the terrain must match, and at most 3 actors. Restores forced-awake, pause, daytime, player character position/rotation. Actors are matched **by position in the list**:

```text
existing = local actors (list order); result = []
FOR EACH saved entry, index i
  IF i < |existing|
    IF filename, skin (when saved) or section config differ from existing[i]
      unseat the player if it was in it (or forget it as previous player); queue deletion of existing[i]
    ELSE reuse existing[i]; hard-reset it in place (SyncReset without repositioning)
  IF not reused AND NOT entry.preloaded_with_terrain
    queue a spawn request (origin SAVEGAME) at (x, min_height, z), rotation 270° − heading about Y,
    with skin, tuneup, section config and a copy of the entry as "saved state"   # restored after spawn
  result += reused actor or none
delete remaining existing actors beyond the saved count (unseating as above)
FOR EACH reused actor: restore_saved_state(actor, entry)
```

Preloaded actors that are not present are silently skipped (they are not installed). "Scene loaded" unless autosave.

## `RestoreSavedState(actor, entry)`

**Contract** — applies an entry to an actor (called directly for reused actors, or after spawn for new ones):

1. spawn heading, simulation state, pause; seat the player (queued) or set previous player;
2. engine: start/stop to match, then push rpm/gear/running/contact/auto mode/auto select as a network-style state; wheel spin; ABS/TC/cruise flags and targets;
3. hydro states, brakes, wheel speeds, custom particles (toggle to match), lights, blinkers, beacons, high beams, fog, custom lights, buoyancy sink;
4. aero engines and screw props (indexed); rotator angles; wheels detached (skidmarks reset); differentials toggled until the active type matches (at most one full rotation); transfer case 4WD and ratio (rotated until the first ratio matches);
5. command key values and per-beam auto-move / centre modes;
6. node positions (relative to the actor origin), velocities, initial positions;
7. beams: stresses, strength, length, broken, disabled, inter-actor; re-link inter-actor beams to the referenced actor (index into the *current* local actor list);
8. hooks / ropes / ties re-attached only when both target indices are valid (hook and tie beams are re-pointed at the lock node / ropable node); ropable counters;
9. slide nodes reset, lock state toggled to match; bounding boxes and average position recomputed (previous = current, so the first velocity is zero).

## Quicksave plumbing (`GameContext`)

- Terrain quicksave name: `quicksave_<terrain without .terrn2>[_mp].sav`.
- Hotkeys: quick-load slots 1–10 (slot 10 = file `quicksave-0.sav`) load `quicksave-<n>.sav` at any time; quick-save slots and the terrain quicksave work only with a running terrain. The terrain quick-load asks for confirmation ("Load game?" / "You will lose all unsaved progress!") unless the user disabled the dialog.
- `ExtractSceneName` / `ExtractSceneTerrain` read just those fields for the load menu (empty on any error).
