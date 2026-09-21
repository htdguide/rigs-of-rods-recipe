# source/main/scripting/GameScript.h

> The `game` object: the general-purpose API scripts use to query and drive the simulation.

**Needs** — [`Application.h`](../Application.h.md) · [Seam: Script engine](../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`GameScript.cpp`](GameScript.cpp.md) · [`ScriptEngine.cpp`](ScriptEngine.cpp.md) · [`ScriptEngine.h`](ScriptEngine.h.md) · [`scripting/bindings/GameScriptAngelscript.cpp`](bindings/GameScriptAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

A façade: nearly every method forwards to one subsystem through the global application context. It exists so scripts get one stable, documented surface that survives internal refactors. Registered as `GameScriptClass game` by [`bindings/GameScriptAngelscript.cpp`](bindings/GameScriptAngelscript.cpp.md). Implementation: [`GameScript.cpp`](GameScript.cpp.md).

## State

Stateless — every call reads live game state.

## Conventions

- Functions that need a loaded terrain, a player character or a camera check first and log "cannot execute, no terrain/character/camera" instead of failing.
- Functions that touch files accept a bare filename plus a resource group; any path component is rejected ("file paths are not allowed").
- Native failures are caught and forwarded as `GENERIC_EXCEPTION_CAUGHT` script events.
- Requests that change game structure (load terrain, spawn/delete actor, back to menu, quit) are posted as messages and happen at the next message-queue pass, not during the call.

## API groups

- **General** — `log`, `getTime` (simulated seconds), `getFPS`, `getAvgFPS`, `rangeRandom`, `backToMenu`, `quitGame`, `openUrlInDefaultBrowser`, `fetchUrlAsStringAsync`, `useOnlineAPI`, `pushMessage(type, dictionary)`.
- **Resources** — `checkResourceExists`, `deleteResource`, `loadTextResourceAsString`, `createTextResourceFromString(overwrite?)`, `findResourceFileInfo(group, pattern, dirs?)`, `loadImageResource`, `serializeMeshResource`, `getSceneManager`.
- **GUI** — `flashMessage`, `message`, `get/setChatFontSize`, `showMessageBox`, `showChooser(type, instance, box)`, `updateDirectionArrow`, `hideDirectionArrow`, `getScreenPosFromWorldPos`, `getDisplaySize`, `getMouseScreenPosition`.
- **Script management** — `registerForEvent`, `unRegisterEvent`, `get/setRegisteredEventsMask(unit)`, `add/delete/…ScriptFunction`, `…ScriptVariable`, `getScriptVariable`, `clearEventCache`, `sendGameCmd`, `getRunningScripts`, `getScriptDetails(unit)`.
- **Terrain** — `loadTerrain`, `getLoadedTerrain`, `getTerrain`, `getCaelumAvailable`, `get/setCaelumTime`, `get/setGravity`, `getGroundHeight`, `get/setWaterHeight`, `spawnObject`, `moveObjectVisuals`, `destroyObject`, `getEditorObjects`, `getMousePositionOnTerrain`, `getMousePointedMovableObjects`.
- **Character** — `get/setPersonPosition`, `movePerson`, `get/setPersonRotation`.
- **Actors** — `activateAllVehicles`, `setTrucksForcedAwake`, `boostCurrentTruck`, `getCurrentTruck`, `getTruckByNum`, `getAllTrucks`, `getCurrentTruckNumber`, `spawnTruck`, `repairVehicle`, `removeVehicle`, `getNumTrucksByFlag`, `getTruckRemotelyReceivingCommands`, next-id queries for actors, free forces and free-beam graphics.
- **AI** — `spawnTruckAI`, `getWaypoints`, `getWaypointsSpeed`, `addWaypoint`, and getters/setters for the AI panel settings (vehicle count, distance, position scheme, speed, per-vehicle name/config/skin, repeat count, mode), `getCurrentTruckAI`, `getTruckAIByNum`.
- **Camera** — set/get position, direction, orientation; yaw/pitch/roll; `cameraLookAt`.
- **Race** — `startTimer(id)`, `stopTimer`, `setTimeDiff`, `setBestLapTime`.
- **Materials** — set ambient/diffuse/specular/emissive colour; set texture name/rotation/scroll/scale on (material, technique, pass, texture unit).
- **Sound** — list/get sound-script templates and instances, `createSoundFromResource`, `createSoundScriptInstance(template, actor id)`.
