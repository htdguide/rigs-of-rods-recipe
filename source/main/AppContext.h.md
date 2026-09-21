# source/main/AppContext.h

> The system-integration object: owns the render window and viewport, runs the startup steps, and is the first receiver of every OS input event.

**Needs** — [`Application.h`](Application.h.md) · [`utils/ForceFeedback.h`](utils/ForceFeedback.h.md) · [Seam: 3D rendering engine](../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine) · [Seam: Windowing and input devices](../../SYSTEM-REQUIREMENTS.md#seam-windowing-and-input-devices)
**Used by** — [`AppContext.cpp`](AppContext.cpp.md) · [`Application.cpp`](Application.cpp.md) · [`GameContext.cpp`](GameContext.cpp.md) · [`gameplay/Engine.cpp`](gameplay/Engine.cpp.md) · [`gameplay/RaceSystem.cpp`](gameplay/RaceSystem.cpp.md) · [`gfx/GfxScene.cpp`](gfx/GfxScene.cpp.md) · [`gfx/GfxWater.cpp`](gfx/GfxWater.cpp.md) · [`gfx/HydraxWater.cpp`](gfx/HydraxWater.cpp.md) · [`gfx/SkyManager.cpp`](gfx/SkyManager.cpp.md) · [`gfx/SkyXManager.cpp`](gfx/SkyXManager.cpp.md) · [`gfx/camera/CameraManager.cpp`](gfx/camera/CameraManager.cpp.md) · [`gui/GUIManager.cpp`](gui/GUIManager.cpp.md) · [`gui/OverlayWrapper.cpp`](gui/OverlayWrapper.cpp.md) · [`gui/panels/GUI_DirectionArrow.cpp`](gui/panels/GUI_DirectionArrow.cpp.md) · [`gui/panels/GUI_GameSettings.cpp`](gui/panels/GUI_GameSettings.cpp.md) · [`gui/panels/GUI_RepositorySelector.cpp`](gui/panels/GUI_RepositorySelector.cpp.md) · [`gui/panels/GUI_SimPerfStats.cpp`](gui/panels/GUI_SimPerfStats.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](gui/panels/GUI_SurveyMap.cpp.md) · [`main.cpp`](main.cpp.md) · [`network/DiscordRpc.cpp`](network/DiscordRpc.cpp.md) · [`physics/ActorSpawner.cpp`](physics/ActorSpawner.cpp.md) · [`physics/water/Wavefield.cpp`](physics/water/Wavefield.cpp.md) · [`resources/terrn2_fileformat/Terrn2FileFormat.cpp`](resources/terrn2_fileformat/Terrn2FileFormat.cpp.md) · [`scripting/GameScript.cpp`](scripting/GameScript.cpp.md) · [`terrain/TerrainEditor.cpp`](terrain/TerrainEditor.cpp.md) · [`utils/InputEngine.cpp`](utils/InputEngine.cpp.md) · [`utils/memory/RefCountingObject.h`](utils/memory/RefCountingObject.h.md)
**Tier floor** — T2

## Purpose

Everything that touches the operating system's window and devices lives behind this one object, so the rest of the game sees a viewport, a main-thread id and pre-routed input. Implementation: [`AppContext.cpp`](AppContext.cpp.md).

## State

```text
RECORD AppContext
  engine_root      : rendering engine root
  render_window    : window
  viewport         : viewport on render_window (camera attached later)
  windowed_fix_done: bool     # see ActivateFullscreen
  profiler_enabled : bool     # last applied profiler state
  last_screenshot_time  : wall-clock seconds
  last_screenshot_index : int # disambiguates screenshots within one second
  force_feedback   : ForceFeedback
  main_thread_id   : thread id
```

## Startup steps

Called by [`main.cpp`](main.cpp.md) in this order: `SetUpThreads`, `SetUpProgramPaths`, `SetUpLogging`, `SetUpResourcesDir`, `SetUpRendering`, `SetUpConfigSkeleton`, `SetUpInput`, `SetUpObsoleteConfMarker`. Each returns failure to abort startup. Details in the `.cpp` twin.

## Rendering helpers

`CreateCustomRenderWindow(name, w, h)` (secondary windows for video cameras), `CaptureScreenshot()`, `ActivateFullscreen(bool)`, `PrepareProfiler()`.

## Getters

Engine root, viewport, render window, force feedback, main-thread id.

## Event receivers

Window resized, window focus changed; mouse moved/pressed/released; key pressed/released; joystick button/axis/slider/POV. Routing rules are in the `.cpp` twin.
