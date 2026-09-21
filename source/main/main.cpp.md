# source/main/main.cpp

> Program entry: ordered startup, the initial requests, and the main loop — message processing, input, scripts, audio, simulation hand-off and rendering, once per frame.

**Needs** — [`physics/Actor.h`](physics/Actor.h.md) · [`Application.h`](Application.h.md) · [`AppContext.h`](AppContext.h.md) · [`resources/CacheSystem.h`](resources/CacheSystem.h.md) · [`gfx/camera/CameraManager.h`](gfx/camera/CameraManager.h.md) · [`gameplay/ChatSystem.h`](gameplay/ChatSystem.h.md) · [`physics/collision/Collisions.h`](physics/collision/Collisions.h.md) · [`system/Console.h`](system/Console.h.md) · [`resources/ContentManager.h`](resources/ContentManager.h.md) · [`network/DiscordRpc.h`](network/DiscordRpc.h.md) · [`utils/ErrorUtils.h`](utils/ErrorUtils.h.md) · [`GameContext.h`](GameContext.h.md) · [`gfx/GfxScene.h`](gfx/GfxScene.h.md) · [`gui/panels/GUI_DirectionArrow.h`](gui/panels/GUI_DirectionArrow.h.md) · [`gui/panels/GUI_FrictionSettings.h`](gui/panels/GUI_FrictionSettings.h.md) · [`gui/panels/GUI_GameControls.h`](gui/panels/GUI_GameControls.h.md) · [`gui/panels/GUI_LoadingWindow.h`](gui/panels/GUI_LoadingWindow.h.md) · [`gui/panels/GUI_MainSelector.h`](gui/panels/GUI_MainSelector.h.md) · [`gui/panels/GUI_MessageBox.h`](gui/panels/GUI_MessageBox.h.md) · [`gui/panels/GUI_MultiplayerSelector.h`](gui/panels/GUI_MultiplayerSelector.h.md) · [`gui/panels/GUI_MultiplayerClientList.h`](gui/panels/GUI_MultiplayerClientList.h.md) · [`gui/panels/GUI_RepositorySelector.h`](gui/panels/GUI_RepositorySelector.h.md) · [`gui/panels/GUI_VehicleInfoTPanel.h`](gui/panels/GUI_VehicleInfoTPanel.h.md) · [`gui/GUIManager.h`](gui/GUIManager.h.md) · [`gui/GUIUtils.h`](gui/GUIUtils.h.md) · [`utils/InputEngine.h`](utils/InputEngine.h.md) · [`utils/Language.h`](utils/Language.h.md) · [`audio/MumbleIntegration.h`](audio/MumbleIntegration.h.md) · [`network/OutGauge.h`](network/OutGauge.h.md) · [`gui/OverlayWrapper.h`](gui/OverlayWrapper.h.md) · [`utils/PlatformUtils.h`](utils/PlatformUtils.h.md) · [`RoRVersion.h`](../../source/version_info/RoRVersion.h.md) · [`scripting/ScriptEngine.h`](scripting/ScriptEngine.h.md) · [`gfx/Skidmark.h`](gfx/Skidmark.h.md) · [`audio/SoundScriptManager.h`](audio/SoundScriptManager.h.md) · [`terrain/Terrain.h`](terrain/Terrain.h.md) · [`utils/Utils.h`](utils/Utils.h.md)
**Used by** — nothing (program entry point)
**Tier floor** — T2


## Purpose

The single place where frame ordering is decided. Every subsystem exposes "update" or "handle" calls; this file fixes the order in which they run, and implements the handler for every game message ([`GameContext.h`](GameContext.h.md)). Understanding this file is understanding how the physics thread, the renderer and the GUI avoid stepping on each other.

## State

Only locals: the frame clock. Everything else lives in the application singletons ([`Application.h`](Application.h.md), [`AppContext`](AppContext.h.md)).

## Startup (order matters)

```text
init HTTP library globally (before any thread starts)
register built-in settings; set up threads; resolve program/user paths (fail → exit −1); logging
derive user subfolders: config, cache, thumbnails, savegames, screenshots, scripts, projects, repo_attachments
load RoR.cfg; apply command line (help/version → print and exit 0)
locate the resources directory; create config folder; start the renderer and window; default 5 mipmaps; config skeleton
overlay system
render targets sized from the video mode (largest power of two ≤ width = res; FSAA rounded down to an even value):
  "EnvironmentTexture" cube res/4, "Refraction" and "Reflection" res/2, RGB8
unless debugging textures: make the renderer's missing-texture placeholder black instead of a checkerboard
add resource packs: flags, fonts, icons, core, wallpapers, scripts; translations; console commands (after translations)
content manager; graphics scene (needs content); camera; environment map (needs camera); GUI (needs scene)
Discord; input; script engine (+ scripts and projects folders); menu wallpaper; obsolete-config marker; thread pool
default inertia models
```

## Initial requests (posted, not executed)

1. Mod cache: purge if forced, update if forced (config or command line), else load.
2. Scripts from the command line, then from `app_custom_scripts`, as CUSTOM.
3. Session: command-line server → connect; else "join on startup" → connect; else a preset terrain (command line, then config) → load it; else resume `autosave.sav` if asked and present; else the terrain selector if "skip main menu"; else the main menu.

Then: state MAIN_MENU, wallpaper shown, menu music (`tracks/main_menu_tune`) if enabled, one forced loading-window frame.

## Main loop (until SHUTDOWN)

```text
pump window events
IF simulating: wait for the physics thread to finish the previous step (sync point)
process every queued message (below); after each successful one, post its chain
FPS limit (if set, clamped 5–240): busy-wait until 1/limit s since the last frame
dt = time since last frame
online: drain received stream packets → chat; if simulating also actors, then characters (last, so seat couplings resolve)
IF dt ≠ 0: capture input, update key debounces; unless capturing a key binding:
  savegame hotkeys (only when no menu-type window is open); global input; GUI hotkeys
  simulating:
    editor mode: sky input, terrain editor input
    else: characters update (before camera), camera input, overlay timers, repair mode, actor-manager input
      running and camera not free: sim input, sky input; in a local (non-remote) vehicle: common vehicle input,
        and unless replaying: truck / aircraft / boat input by type; prop animation input for it and linked actors
      free camera: sky input only
OutGauge (if enabled)
new GUI frame; simulating: sim-synced GUI, debug-view updates, vehicle panel stats, friction panel ground
Mumble; sound listener and sources; scripts' frameStep; force feedback (if enabled and running)
running: scene-mouse physics
running/paused/editor: copy simulation state into the graphics buffers
dt_sim = dt × simulation speed while running and not physics-paused, else 0
running: water wave step; start the next physics step on the physics thread (no reading actor physics after this)
draw: main menu GUI, or the scene (which also draws the buffered GUI) with dt_sim
window closed → post shutdown; else render one frame (also when inactive but visible)
apply GUI keyboard capture; mouse-cursor auto-hide
```

Unhandled renderer or runtime exceptions (release builds) show an error dialog and end the program.

## Message processing

Each handler catches exceptions and reports them (the message counts as done). Handlers that cannot proceed yet **re-post themselves** and mark themselves failed so their chain waits. Payloads are owned by the message and freed by the handler.

| Message | Handling |
|---|---|
| APP_SHUTDOWN | in simulation save `autosave.sav`; save config; stop Discord; disconnect; state SHUTDOWN; disable script events (fast exit) |
| APP_SCREENSHOT | capture with the cursor hidden |
| APP_DISPLAY_FULLSCREEN / WINDOWED | switch, with a notice |
| APP_MODCACHE_LOAD / UPDATE / PURGE | load (if not yet loaded), update or rebuild the cache — update/purge only in the main menu (actors hold cache entries) |
| APP_LOAD_SCRIPT / UNLOAD_SCRIPT | via the script engine (actor resolved by instance id) |
| APP_SCRIPT_THREAD_STATUS | forward as `ANGELSCRIPT_THREAD_STATUS` event |
| APP_REINIT_INPUT | recreate the input system; hide the notice |
| NET_CONNECT_REQUESTED / DISCONNECT_REQUESTED | start connecting / disconnect (in the menu also close the selector and reopen the menu) |
| NET_SERVER_KICK, NET_RECV_ERROR | disconnect, unload terrain, open menu, message box with the reason |
| NET_CONNECT_STARTED / PROGRESS | loading window with status (and close MP selector, menu) |
| NET_CONNECT_SUCCESS | stop the connect thread; CONNECTED; register the chat stream; create Mumble link; load the server's terrain, or if it is "any" the preset terrain or the terrain selector |
| NET_CONNECT_FAILURE | disconnect, menu, "connection failed" box |
| NET_REFRESH_SERVERLIST_*, REPOLIST_*, OPEN_RESOURCE_SUCCESS, DOWNLOAD_REPOIMAGE_*, DOWNLOAD_REPOFILE_* | hand results to the multiplayer and repository panels; file download progress drives the loading window |
| NET_FETCH_AI_PRESETS_* | store parsed external presets or the error; refresh the list |
| NET_ADD / REMOVE_PEEROPTIONS | update the peer's options; muting chat purges their chat lines; mute/hide/unmute/unhide their actors via messages |
| SIM_PAUSE / UNPAUSE | mute all actor sounds + PAUSED / unmute (except peer-muted) + RUNNING |
| SIM_LOAD_TERRN | gameplay resources; load terrain; on success: player character, preset vehicle, scene mouse, overlays, direction arrow, stop menu music, ambient light 0.7 (sandstorm sky) or 0.3, Discord presence, RUNNING + SIMULATION, hide menu/wallpaper/loading, restore default FOVs, online chat hint, connect OutGauge. Failure: disconnect (online) or menu; failed |
| SIM_UNLOAD_TERRN | editor: write editor log; save `autosave.sav`; leave vehicle; clean up simulation, characters, scene mouse, overlays, cameras, debug views; close selector; wallpaper; clear AI waypoints; OFF + MAIN_MENU; unload terrain; clear scene; forget terrain names; close OutGauge; reset listener; release sound sources; reset race UI |
| SIM_LOAD_SAVEGAME | unreadable → error (and menu); same terrain → load scene; different terrain online → "terrain mismatch"; else unload (if any), load the savegame's terrain, then (chained) this message again |
| SIM_SPAWN / MODIFY / DELETE_ACTOR, SEAT_PLAYER, TELEPORT_PLAYER | only while simulating; see [`GameContext.cpp`](GameContext.cpp.md) |
| SIM_HIDE / UNHIDE_NET_ACTOR | online remote actors only: toggle NETWORKED_OK ↔ HIDDEN, visuals, shadows, sounds, flares off, smoke |
| SIM_MUTE / UNMUTE_NET_ACTOR | remote actors: peer-mute flag and sounds |
| SIM_SCRIPT_EVENT_TRIGGERED | engine side effects first (broken free force → remove its beam graphics; new cache entry online → retry failed remote stream registrations), then the script event |
| SIM_SCRIPT_CALLBACK_QUEUED | eventbox callback into scripts |
| SIM_ACTOR_LINKING | hook lock/unlock/toggle, mouse hook (+ `TRUCK_MOUSE_GRAB`), tie, rope, slide-node lock |
| SIM_ADD / MODIFY / REMOVE_FREEFORCE | actor manager (removal also drops graphics) |
| GUI_OPEN / CLOSE_MENU, OPEN / CLOSE_SELECTOR, MP_CLIENTS_REFRESH, SHOW_MESSAGE_BOX, REFRESH_TUNING_MENU, SHOW_CHATBOX (optional prefill), OPEN_MP_SETTINGS | the corresponding panel action |
| EDI_MODIFY_GROUNDMODEL | copy the edited ground model over the live one with the same name |
| EDI_ENTER / LEAVE_TERRN_EDITOR, SAVE_TERRN_CHANGES | editor mode on (with hints) / off (write log, deselect) / write edits to `.tobj` files (unpacked terrains only) |
| EDI_LOAD_BUNDLE | load; `MODCACHE_ACTIVITY(BUNDLE_LOADED)` |
| EDI_RELOAD_BUNDLE | delete actors using its resource group first (re-post until none), then reload; `BUNDLE_RELOADED` |
| EDI_UNLOAD_BUNDLE | delete actors using it as vehicle, skin, addon part or asset pack; if it is the terrain, disconnect/unload; re-post until clear, then unload; `BUNDLE_UNLOADED` |
| EDI_DELETE_BUNDLE | if installed: unload every loaded entry from that archive (re-post until clear), then delete the archive and tell the repository panel |
| EDI_CREATE / MODIFY / DELETE_PROJECT | cache-system projects (modify refused online) |
| EDI_ADD / MODIFY / DELETE_FREEBEAMGFX | graphics scene |

**Notes** — the "retry by re-posting" handlers rely on the actor deletions they queued being processed first; since both go to the back of the same queue, a reload/unload message cycles once per frame at most until the actors are gone.
