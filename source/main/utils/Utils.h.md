# source/main/utils/Utils.h

> Grab-bag of string, hashing, UTF-8, rounding and projection helpers.

**Needs** — [`Application.h`](../Application.h.md) · [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine)
**Used by** — [`GameContext.cpp`](../GameContext.cpp.md) · [`audio/SoundScriptManager.cpp`](../audio/SoundScriptManager.cpp.md) · [`gameplay/Character.cpp`](../gameplay/Character.cpp.md) · [`gameplay/CharacterFactory.cpp`](../gameplay/CharacterFactory.cpp.md) · [`gameplay/ChatSystem.cpp`](../gameplay/ChatSystem.cpp.md) · [`gameplay/Replay.cpp`](../gameplay/Replay.cpp.md) · [`gameplay/TorqueCurve.cpp`](../gameplay/TorqueCurve.cpp.md) · [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`gfx/GfxScene.cpp`](../gfx/GfxScene.cpp.md) · [`gfx/Skidmark.cpp`](../gfx/Skidmark.cpp.md) · [`gui/DashBoardManager.cpp`](../gui/DashBoardManager.cpp.md) · [`gui/GUIUtils.cpp`](../gui/GUIUtils.cpp.md) · [`gui/OverlayWrapper.cpp`](../gui/OverlayWrapper.cpp.md) · [`gui/panels/GUI_CollisionsDebug.cpp`](../gui/panels/GUI_CollisionsDebug.cpp.md) · [`gui/panels/GUI_FlexbodyDebug.cpp`](../gui/panels/GUI_FlexbodyDebug.cpp.md) · [`gui/panels/GUI_FrictionSettings.cpp`](../gui/panels/GUI_FrictionSettings.cpp.md) · [`gui/panels/GUI_GameAbout.cpp`](../gui/panels/GUI_GameAbout.cpp.md) · [`gui/panels/GUI_MainSelector.cpp`](../gui/panels/GUI_MainSelector.cpp.md) · [`gui/panels/GUI_ScriptMonitor.cpp`](../gui/panels/GUI_ScriptMonitor.cpp.md) · [`gui/panels/GUI_TextureToolWindow.cpp`](../gui/panels/GUI_TextureToolWindow.cpp.md) · [`gui/panels/GUI_VehicleInfoTPanel.cpp`](../gui/panels/GUI_VehicleInfoTPanel.cpp.md) · [`main.cpp`](../main.cpp.md) · [`network/Network.cpp`](../network/Network.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/CmdKeyInertia.cpp`](../physics/CmdKeyInertia.cpp.md) · [`physics/Savegame.cpp`](../physics/Savegame.cpp.md) · [`physics/flex/FlexBody.h`](../physics/flex/FlexBody.h.md) · [`resources/CacheSystem.cpp`](../resources/CacheSystem.cpp.md) · [`resources/ContentManager.cpp`](../resources/ContentManager.cpp.md) · [`resources/odef_fileformat/ODefFileFormat.cpp`](../resources/odef_fileformat/ODefFileFormat.cpp.md) · [`resources/otc_fileformat/OTCFileFormat.cpp`](../resources/otc_fileformat/OTCFileFormat.cpp.md) · [`resources/rig_def_fileformat/RigDef_Parser.cpp`](../resources/rig_def_fileformat/RigDef_Parser.cpp.md) · [`resources/skin_fileformat/SkinFileFormat.cpp`](../resources/skin_fileformat/SkinFileFormat.cpp.md) · [`resources/terrn2_fileformat/Terrn2FileFormat.cpp`](../resources/terrn2_fileformat/Terrn2FileFormat.cpp.md) · [`resources/tuneup_fileformat/TuneupFileFormat.cpp`](../resources/tuneup_fileformat/TuneupFileFormat.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`scripting/OgreScriptBuilder.cpp`](../scripting/OgreScriptBuilder.cpp.md) · [`scripting/ScriptEngine.cpp`](../scripting/ScriptEngine.cpp.md) · [`system/AppCommandLine.cpp`](../system/AppCommandLine.cpp.md) · [`system/AppConfig.cpp`](../system/AppConfig.cpp.md) · [`system/Console.cpp`](../system/Console.cpp.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md) · [`ConfigFile.cpp`](ConfigFile.cpp.md) · [`ErrorUtils.cpp`](ErrorUtils.cpp.md) · [`Utils.cpp`](Utils.cpp.md)
**Tier floor** — T4

## Purpose

Shared helpers with no common theme. Declarations; behaviour in [`Utils.cpp`](Utils.cpp.md). Two small inline pieces are defined here.

## State

Stateless.

## `replaceString(str, search, replacement)`

**Contract** — replaces every occurrence in place, scanning left to right; after each replacement the scan resumes **one character after the start of the replacement** (not after its end), so a replacement containing the search text can be re-matched from its second character.

## `EraseIf(list, predicate)`

**Contract** — removes all elements satisfying the predicate, preserving the order of the rest.

## `World2ScreenConverter`

**Contract** — constructed from a view matrix, projection matrix and screen size; `Convert(world_pos)` returns `(screen_x, screen_y, view_z)` where screen y grows downward and `view_z < 0` means the point is in front of the camera. Used to place 2D labels (player names, node ids) over 3D objects.

```text
FUNCTION convert(world)
  v = view * world
  c = projection * v                      # clip coords in [-1, 1]
  sx = (c.x / 2 + 0.5) * screen.w
  sy = (1 - (c.y / 2 + 0.5)) * screen.h
  RETURN (sx, sy, v.z)
```

## Other declarations

`sha1sum`, `HashData`, `tryConvertUTF`, `formatBytes`, `getTimeStamp`, `getVersionString`, `Round`, `SanitizeUtf8String`, `SanitizeUtf8CString`, `Utf8ToWideChar`, `TrimStr`, `Sha1Hash`, `JoinStrVec`, `IsDistanceWithin`, `PrintMeshInfo`, `CvarAddFileToList`, `CvarRemoveFileFromList`, `SplitBundleQualifiedFilename` — see the `.cpp` twin.
