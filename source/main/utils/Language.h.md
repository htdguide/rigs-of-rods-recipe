# source/main/utils/Language.h

> Translation lookup macros and the list of installed languages.

**Needs** — [`Application.h`](../Application.h.md) · [Seam: Localization catalogs](../../../SYSTEM-REQUIREMENTS.md#seam-localization-catalogs)
**Used by** — [`Application.cpp`](../Application.cpp.md) · [`gameplay/ChatSystem.cpp`](../gameplay/ChatSystem.cpp.md) · [`gameplay/Landusemap.cpp`](../gameplay/Landusemap.cpp.md) · [`gameplay/Replay.cpp`](../gameplay/Replay.cpp.md) · [`gfx/camera/CameraManager.cpp`](../gfx/camera/CameraManager.cpp.md) · [`gui/GUIManager.cpp`](../gui/GUIManager.cpp.md) · [`gui/OverlayWrapper.cpp`](../gui/OverlayWrapper.cpp.md) · [`gui/panels/GUI_CollisionsDebug.cpp`](../gui/panels/GUI_CollisionsDebug.cpp.md) · [`gui/panels/GUI_ConsoleView.cpp`](../gui/panels/GUI_ConsoleView.cpp.md) · [`gui/panels/GUI_ConsoleWindow.cpp`](../gui/panels/GUI_ConsoleWindow.cpp.md) · [`gui/panels/GUI_DirectionArrow.cpp`](../gui/panels/GUI_DirectionArrow.cpp.md) · [`gui/panels/GUI_FlexbodyDebug.cpp`](../gui/panels/GUI_FlexbodyDebug.cpp.md) · [`gui/panels/GUI_FrictionSettings.cpp`](../gui/panels/GUI_FrictionSettings.cpp.md) · [`gui/panels/GUI_GameAbout.cpp`](../gui/panels/GUI_GameAbout.cpp.md) · [`gui/panels/GUI_GameChatBox.cpp`](../gui/panels/GUI_GameChatBox.cpp.md) · [`gui/panels/GUI_GameControls.cpp`](../gui/panels/GUI_GameControls.cpp.md) · [`gui/panels/GUI_GameMainMenu.cpp`](../gui/panels/GUI_GameMainMenu.cpp.md) · [`gui/panels/GUI_GameSettings.cpp`](../gui/panels/GUI_GameSettings.cpp.md) · [`gui/panels/GUI_LoadingWindow.cpp`](../gui/panels/GUI_LoadingWindow.cpp.md) · [`gui/panels/GUI_MainSelector.cpp`](../gui/panels/GUI_MainSelector.cpp.md) · [`gui/panels/GUI_MessageBox.cpp`](../gui/panels/GUI_MessageBox.cpp.md) · [`gui/panels/GUI_MultiplayerClientList.cpp`](../gui/panels/GUI_MultiplayerClientList.cpp.md) · [`gui/panels/GUI_MultiplayerSelector.cpp`](../gui/panels/GUI_MultiplayerSelector.cpp.md) · [`gui/panels/GUI_NodeBeamUtils.cpp`](../gui/panels/GUI_NodeBeamUtils.cpp.md) · [`gui/panels/GUI_RepositorySelector.cpp`](../gui/panels/GUI_RepositorySelector.cpp.md) · [`gui/panels/GUI_SimPerfStats.cpp`](../gui/panels/GUI_SimPerfStats.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](../gui/panels/GUI_SurveyMap.cpp.md) · [`gui/panels/GUI_TextureToolWindow.cpp`](../gui/panels/GUI_TextureToolWindow.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`gui/panels/GUI_VehicleInfoTPanel.cpp`](../gui/panels/GUI_VehicleInfoTPanel.cpp.md) · [`main.cpp`](../main.cpp.md) · [`network/Network.cpp`](../network/Network.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/Differentials.cpp`](../physics/Differentials.cpp.md) · [`physics/Savegame.cpp`](../physics/Savegame.cpp.md) · [`physics/collision/Collisions.cpp`](../physics/collision/Collisions.cpp.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`resources/ContentManager.cpp`](../resources/ContentManager.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`system/AppConfig.cpp`](../system/AppConfig.cpp.md) · [`system/ConsoleCmd.h`](../system/ConsoleCmd.h.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md) · [`terrain/TerrainGeometryManager.cpp`](../terrain/TerrainGeometryManager.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md) · [`ErrorUtils.cpp`](ErrorUtils.cpp.md) · [`InputEngine.cpp`](InputEngine.cpp.md) · [`Language.cpp`](Language.cpp.md)
**Tier floor** — T4

## Purpose

Every user-visible string passes through one of two lookups: `_L(text)` (plain) and `_LC(context, text)` (gettext message context — lets the same English word translate differently in different places). The source text is the key; a missing translation returns the key. Implementation: [`Language.cpp`](Language.cpp.md).

## State

```text
RECORD LanguageEngine
  languages : list<(display_name, code)>   # first entry always ("English", "en")
```

## `setup()`

See the `.cpp` twin.

## `getLanguages()`

**Contract** — the discovered list, for the settings UI.
