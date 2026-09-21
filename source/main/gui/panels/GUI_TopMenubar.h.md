# source/main/gui/panels/GUI_TopMenubar.h

> The hover-activated menu bar at the top of the screen during simulation, plus the centred state box (pause, replay, race, repair, editor).

**Needs** — [`resources/addonpart_fileformat/AddonPartFileFormat.h`](../../resources/addonpart_fileformat/AddonPartFileFormat.h.md) · [`resources/CacheSystem.h`](../../resources/CacheSystem.h.md) · [`network/RoRnet.h`](../../network/RoRnet.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`GameContext.cpp`](../../GameContext.cpp.md) · [`gameplay/VehicleAI.cpp`](../../gameplay/VehicleAI.cpp.md) · [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_TopMenubar.cpp`](GUI_TopMenubar.cpp.md) · [`network/Network.cpp`](../../network/Network.cpp.md) · [`physics/ActorManager.cpp`](../../physics/ActorManager.cpp.md) · [`scripting/GameScript.cpp`](../../scripting/GameScript.cpp.md)
**Tier floor** — T2


## Purpose

Most in-game actions live here. It also owns the AI traffic settings (read by the `AI.as` script through the `game` API) and the tuning menu state. Implementation: [`GUI_TopMenubar.cpp`](GUI_TopMenubar.cpp.md).

## State

```text
RECORD ai_events: position, speed (−1 = default)
ENUM TopMenu: NONE, SIM, ACTORS, SAVEGAMES, SETTINGS, TOOLS, AI, TUNING
ENUM StateBox: NONE, REPLAY, RACE, LIVE_REPAIR, QUICK_REPAIR, IMPORT_TERRAIN, OVERWRITE_TERRAIN
CONSTANTS: menu y offset 40; top hover band 50 px; menu hover padding 25×10; hold-to-confirm 1.5 s
RECORD TopMenubar
  AI: waypoints; count 1; speed 50; repeat 1; altitude 1000 ft; distance 20 m; position scheme (behind | parallel | opposite);
      vehicle 1 and 2 (file, display name, config, skin; default "95bbUID-agoras.truck" / "Bus RVI Agora S");
      selecting vehicle 1/2; recording; mode (normal | race | drag race | crash | chase); keep-menu-open;
      saved values to restore after drag/crash modes
  AI presets: bundled (from the terrain), external (downloaded or local file), merged list; fetching flag; error text
  tuning: subject vehicle; eligible addon parts; per part "conflicts with a used part"; conflict list; saved tune-ups;
          save box (name, visible, overwrite); hovered addon part; right-widget minimum x
  open menu + hover box; state box + hover box; confirm remove-all; daytime at open; waves height;
  quickload available; terrain import started; quicksave name; save slot names
```

## API

`Draw(dt)`, `ShouldDisplay(pos)`, `IsVisible`, `FetchExternAiPresetsOnBackground`, `LoadBundledAiPresets(terrain)`, `RefreshAiPresets`, `RefreshTuningMenu`.
