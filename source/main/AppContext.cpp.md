# source/main/AppContext.cpp

> Startup sequence for paths, logging, renderer and input; input routing between GUI, legacy overlays, camera and scene; screenshots and fullscreen.

**Needs** — [`AppContext.h`](AppContext.h.md) · [`gfx/AdvancedScreen.h`](gfx/AdvancedScreen.h.md) · [`physics/Actor.h`](physics/Actor.h.md) · [`gfx/camera/CameraManager.h`](gfx/camera/CameraManager.h.md) · [`system/Console.h`](system/Console.h.md) · [`gui/GUIManager.h`](gui/GUIManager.h.md) · [`utils/InputEngine.h`](utils/InputEngine.h.md) · [`utils/PlatformUtils.h`](utils/PlatformUtils.h.md) · [`utils/ErrorUtils.h`](utils/ErrorUtils.h.md) · [`gui/OverlayWrapper.h`](gui/OverlayWrapper.h.md) · [`GameContext.h`](GameContext.h.md) · [`../version_info/RoRVersion.h`](../version_info/RoRVersion.h.md) · [Seam: 3D rendering engine](../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine) · [Seam: Windowing and input devices](../../SYSTEM-REQUIREMENTS.md#seam-windowing-and-input-devices) · [Seam: Immediate-mode GUI](../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`AppContext.h`](AppContext.h.md) (see its Used by)
**Tier floor** — T2

## Purpose

Implements [`AppContext.h`](AppContext.h.md). The interesting decisions are the directory layout at startup, how the renderer configuration is recovered when broken, and the **input priority chain**.

## State

See the header twin.

## `SetUpProgramPaths`

**Contract** — sets `sys_process_dir` to the executable's directory and decides `sys_user_dir`. Fails (with a user-visible error) if either path cannot be determined.

```text
FUNCTION set_up_program_paths()
  process_dir = parent directory of the running executable
  IF directory process_dir/"config" EXISTS
    user_dir = process_dir/"config"          # portable installation
  ELSE
    home = user's home (Documents folder on Windows)
    user_dir = CASE platform OF
      Windows: home/"My Games"/"Rigs of Rods"   (create both levels)
      Linux:   $SNAP_USER_COMMON if set, else home/".rigsofrods"
      macOS:   home/"RigsOfRods"
    CREATE user_dir
```

## `SetUpLogging`

**Contract** — `sys_logs_dir = user_dir/logs` (created); opens `RoR.log` as the default log (also echoed to stdout); writes the version line and today's date; subscribes the [console](system/Console.h.md) to every log line.

## `SetUpResourcesDir`

**Contract** — `sys_resources_dir = process_dir/resources`; on Linux falls back to `/usr/share/rigsofrods/resources/`. Fails if absent.

## `SetUpRendering`

**Contract** — creates the rendering engine with config file `config/ogre.cfg` and log `RoR.log`, loads renderer plugins listed in `plugins.cfg` next to the executable (see [`plugins.cfg.in`](plugins.cfg.in.md)), restores or repairs the renderer configuration, and opens the main window.

```text
FUNCTION set_up_rendering()
  root = create engine(config = config_dir/"ogre.cfg", log = logs_dir/"RoR.log")
  FOR EACH plugin IN plugins.cfg["Plugin"]  (folder = plugins.cfg["PluginFolder"] or process_dir)
    TRY load plugin; on failure ignore (engine logs it)
  IF plugins.cfg missing: SHOW error "make sure the game is installed correctly"; FAIL
  autodetect = false
  IF NOT root.restore_config()
    autodetect = true
    root.render_system = first available renderer, or SHOW error and FAIL
  IF app_rendersys_override names an installed renderer different from current
    switch to it and save config                    # user picked it in Settings last session
  app_rendersys_override = ""
  root.initialise(no auto window)
  LOG every renderer option with its current and possible values
  (w, h) = parse "W x H" from option "Video Mode"; clamp to at least 800x600
  IF autodetect
    choose the LAST listed video mode that is >= (w, h) in both axes; save config
  window = create window "Rigs of Rods version <ver>", (w, h), fullscreen = option "Full Screen" == "Yes",
           params { FSAA, vsync, sRGB gamma, border fixed unless diag_allow_window_resize,
                    on Windows: monitor index from "Rendering Device", window procedure }
  register window-event listener; set window icon (Windows resource id 101)
  viewport = window.add_viewport(no camera), black background
```

## `SetUpConfigSkeleton`

**Contract** — seeds the user directory from `resources/skeleton.zip`: creates every directory in the archive under `user_dir`, then copies every non-empty file that does not already exist there. Never overwrites user files. Fails if the archive is empty or missing.

**Notes** — this is how `config/`, `mods/`, `vehicles/`, `terrains/`, `cache/`, `screenshots/`… come to exist, and how default `RoR.cfg`, `input.map`, `ground_models.cfg` etc. are installed on first run.

## `SetUpInput`

**Contract** — creates the [`InputEngine`](utils/InputEngine.h.md), registers this object as the mouse, keyboard and joystick listener, and initialises force feedback if `io_ffb_enabled`.

## `SetUpObsoleteConfMarker`

**Contract** — Windows only: if the pre-2017 user folder `<home>\Rigs of Rods 0.4` exists and has no `OBSOLETE_FOLDER.txt`, writes that file (telling the user where mods moved) and shows a one-time message box.

## `SetUpThreads`

**Contract** — records the current thread as the main thread, then creates the placeholder "dummy" cache entry the selector uses — which must happen after the id is known, because reference-counted objects assert main-thread use.

## Input routing

Every event is first given to the immediate-mode GUI; the GUI's "wants mouse/keyboard" answer decides who gets it next.

```text
ON mouse_moved(e)
  gui.wake_up(); gui.inject_mouse_move(e); input_engine.record_motion(e)
  IF NOT gui.wants_mouse
    IF NOT overlays.handle_mouse_moved()           # legacy aircraft/autopilot panel
      IF NOT camera_manager.handle_mouse_moved()
        scene_mouse.handle_mouse_moved()           # grabbing nodes with the mouse

ON mouse_pressed(e, button)
  gui.wake_up(); gui.set_button(button, down); input_engine.record_press(e)
  IF NOT gui.wants_mouse AND NOT overlays.handle_mouse_pressed()
     AND app_state == SIMULATION
    scene_mouse.handle_mouse_pressed(); camera_manager.handle_mouse_pressed()

ON mouse_released(e, button)  # same chain; camera not notified

ON key_pressed(e)
  gui.inject_key_down(e)
  IF NOT gui.keyboard_capture_requested AND NOT gui.wants_keyboard
    input_engine.key_press(e)

ON key_released(e)
  gui.inject_key_up(e)
  IF NOT gui.keyboard_capture_requested AND NOT gui.wants_keyboard
    input_engine.key_release(e)
  ELSE IF input_engine.key_is_down(e.key)
    input_engine.key_release(e)    # a key pressed before the GUI took focus must still be released

ON any joystick event: input_engine.process_joystick(e)   # the GUI never captures joysticks
```

**Notes** — the "still release keys the game saw go down" rule prevents stuck controls (e.g. throttle held forever) when a text box opens mid-press.

## Window events

- **Resized** — update the input engine's mouse clipping area, the legacy overlays, and (in simulation) every actor's dashboard layout.
- **Focus changed** — clear all GUI mouse-button states and all input-engine key and button states. The device layer does not report releases that happen while the window is unfocused, so without this a key or button held during Alt+Tab stays "down".

## `CaptureScreenshot`

**Contract** — writes `screenshot_<YYYY-MM-DD_HH-MM-SS>_<n>.<format>` into `sys_screenshot_dir`, where `n` restarts at 1 each new second and increments for repeated shots within the same second. For PNG, text chunks are embedded (see [`AdvancedScreen.h`](gfx/AdvancedScreen.h.md)): `User_NickName`, `User_Language`, and when available `Truck_file`, `Truck_name`, `Terrn_file`, `Terrn_name`, `MP_ServerName`. Other formats are the plain window contents. Announces the file name on the console.

## `ActivateFullscreen`

**Contract** — switch the window between fullscreen and windowed at its current size.

**Notes** — the first fullscreen→windowed switch is done as two resizes (−1 px, then +1 px) because the rendering engine misaligns the viewport otherwise; this is a workaround a rebuild on another engine can drop.

## `PrepareProfiler`

**Contract** — when engine profiling is compiled in: apply `diag_profiler_enabled` only when it changed (repeated enables restart the profiler in the engine), and apply `diag_profiler_rate`, resetting it to 10 with a console warning if below 1.

## `CreateCustomRenderWindow`

**Contract** — opens an extra, non-fullscreen window with the main window's anti-aliasing setting (used when video cameras are shown in their own windows).
