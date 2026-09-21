# source/main/scripting/GameScript.cpp

> Implementations of the `game` API; most are forwards, a few carry real rules.

**Needs** — [`GameScript.h`](GameScript.h.md) · [`ScriptUtils.h`](ScriptUtils.h.md) · [`AppContext.h`](../AppContext.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`gameplay/Character.h`](../gameplay/Character.h.md) · [`gameplay/ChatSystem.h`](../gameplay/ChatSystem.h.md) · [`physics/collision/Collisions.h`](../physics/collision/Collisions.h.md) · [`system/Console.h`](../system/Console.h.md) · [`network/CurlHelpers.h`](../network/CurlHelpers.h.md) · [`gameplay/Engine.h`](../gameplay/Engine.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`gui/panels/GUI_TopMenubar.h`](../gui/panels/GUI_TopMenubar.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md) · [`network/Network.h`](../network/Network.h.md) · [`RoRVersion.h`](../../../source/version_info/RoRVersion.h.md) · [`ScriptEngine.h`](ScriptEngine.h.md) · [`gfx/SkyManager.h`](../gfx/SkyManager.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`terrain/TerrainGeometryManager.h`](../terrain/TerrainGeometryManager.h.md) · [`terrain/TerrainObjectManager.h`](../terrain/TerrainObjectManager.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [`gameplay/VehicleAI.h`](../gameplay/VehicleAI.h.md) · [`gfx/GfxWater.h`](../gfx/GfxWater.h.md) · [Seam: Script engine](../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — callers of [`GameScript.h`](GameScript.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GameScript.h`](GameScript.h.md). Only calls with behaviour beyond forwarding are described here; the rest do exactly what their name says to the subsystem named in the header's groups.

## State

Stateless.

## `registerForEvent(mask)` / `unRegisterEvent(mask)`

OR (or clear) the bits into the eventMask of the *currently executing* unit — so a script subscribes itself, typically from `main()`. Outside script execution they do nothing. `get/setRegisteredEventsMask(unit)` read/replace any unit's mask (0 for unknown units).

## `pushMessage`

**Contract** — `pushMessage(type, dictionary) → accepted`.


Lets scripts post game messages, translating dictionary entries into the typed payload the handler expects. Integers arrive as 64-bit.

```text
refused ("not allowed"): modcache load, all network-connection and server/repo list result messages, repo download results
  and progress, message box, selector, ground-model edit
LOAD_SCRIPT      filename and/or buffer (one required), category, associated_actor (required for ACTOR)
UNLOAD_SCRIPT    id
LOAD_TERRN, LOAD_SAVEGAME   filename
SPAWN_ACTOR      filename, position (vector3), rotation (quaternion) required; must be installed (partial name match);
                 instance_id, config (unknown → warning, cleared), first config used by default, enter (bool),
                 skin (unknown → warning, spawned unskinned)
MODIFY_ACTOR     type, instance_id
DELETE_ACTOR, HIDE_NET_ACTOR, UNHIDE_NET_ACTOR   instance_id → the actor (unknown → error)
SEAT_PLAYER      instance_id; −1 or unknown = leave the vehicle
TELEPORT_PLAYER  position
ADD/MODIFY_FREEFORCE  id, type, force_magnitude, base_actor, base_node, then per type:
                 CONSTANT: force_const_direction; TOWARDS_COORDS: target_coords; TOWARDS_NODE: target_actor, target_node;
                 HALFBEAM_GENERIC/ROPE: target_actor, target_node, optional halfb_spring/damp/deform/strength/diameter
REMOVE_FREEFORCE id
LOAD/RELOAD/UNLOAD_BUNDLE  cache_entry
CREATE_PROJECT   name, source_entry
ADD/MODIFY_FREEBEAMGFX  id, freeforce_primary; optional freeforce_secondary, mesh_name, material_name, diameter
DELETE_FREEBEAMGFX id
DOWNLOAD_REPOFILE  resource_id, file_id, filename
missing required key → logged, not posted; other types pass through with no payload
```

**Notes** — in the original an invalid free-force type frees the request and then posts it anyway (dangling payload). A rebuild rejects the message.

## `useOnlineAPI`

**Contract** — `useOnlineAPI(query, dictionary, out result) → 0 | 1 | 2`.


Disabled by config → 0. Not called from a script → 2. No player vehicle → 1. Otherwise build JSON: user-name, user-country, user-token (SHA-1 of the player name), terrain-name, terrain-filename, script-name, script-hash, actor-name/filename/hash, linked-actors [{actor-name, actor-filename, actor-hash}], avg-fps, ror-version, plus every dictionary entry as a string. POST it on a detached thread to `<mp_api_url><query>` with JSON accept/content-type and headers `RoR-Api-User` / `RoR-Api-User-Token`. Result is logged only; returns 0 immediately. Used for race-result submission.

## `fetchUrlAsStringAsync(url, display name)`

Detached GET via [`network/CurlHelpers`](../network/CurlHelpers.h.md); the result arrives as an `APP_SCRIPT_THREAD_STATUS` message which surfaces to scripts as an event with status, HTTP code, transport code, body.

## `spawnObject(object, instance, pos, rot, handler, uniquifyMaterials)`

Needs a terrain with an object manager **and** a terrain script. A non-empty handler name is obsolete: log a deprecation notice and resolve `void <handler>(int, string, string, int)` in the terrain unit (required → warn if missing). Then place the object through the terrain object manager with that handler id.

## Actors

- `getNumTrucksByFlag(state)` — count of actors in that simulation state; 0 counts all.
- `spawnTruck(file, pos, rot°)` — immediate spawn; rotation composed X then Y then Z from degrees.
- `spawnTruckAI(file, pos, config, skin, x)` — heading from the first two AI waypoints (first−second, flattened), or toward the only one; in AI mode 3 ("crash driving") the second vehicle (`x == 1`) uses reversed waypoints. Origin marked AI.
- `boostCurrentTruck(f)` — player engine RPM += 2000·f.
- `repairVehicle(instance, box, keep position)` / `removeVehicle(instance, box)` — act on the actor inside that collision box (eventbox scripting).

## AI waypoints

Waypoints and speeds live in the top-menu AI panel; `getWaypoints(x)` reverses them for `x == 1` in mode 3.

**Notes** — `addWaypoint` in the original copies the list and discards the copy: it has no effect. A rebuild appends to the panel's list.

## `showChooser(type, instance, box)`

Maps "airplane"/"heli" → airplane list, "all", "boat", "car", "extension", "load", "trailer", "train", "truck", "vehicle" → their vehicle-type filter; unknown → nothing. Opens the vehicle selector bound to that eventbox.

## Material texture helpers

`getTextureUnitState(material, technique, pass, unit)` → 1 no material, 2 bad technique, 3 bad pass, 4 bad unit, 0 ok; setters return that code. The original's bounds checks allow index == count (off by one); a rebuild uses `<`.

## Resource file access

`CheckFileAccess(filename)` splits path/base/extension; any path → console error, denied. `loadTextResourceAsString` reads the whole stream (in 4 KB chunks on Linux to work around a short-read bug); `createTextResourceFromString` creates (optionally overwriting) and writes; `findResourceFileInfo` returns dictionaries {filename, basename, compressedSize, uncompressedSize}.

## Screen helpers

`getScreenPosFromWorldPos` projects with the camera view/projection into GUI pixel space; visible only when the point is in front (view-space z < 0). `getMousePositionOnTerrain` ray-casts the mouse into the terrain height field; `getMousePointedMovableObjects` returns every scene object on that ray, nearest first.

## Script inspection

`getRunningScripts` → unit ids; `getScriptDetails(id)` → {uniqueId, scriptName, scriptCategory, eventMask, scriptBuffer} or none. `sendGameCmd(text)` → MSG2_GAME_CMD to the server when connected (0), else −11.
