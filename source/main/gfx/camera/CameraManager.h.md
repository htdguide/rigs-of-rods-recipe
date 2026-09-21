# source/main/gfx/camera/CameraManager.h

> The player camera: behaviours (character, static, vehicle orbit, rail spline, cinecam, free, fixed) and switching rules.

**Needs** — [`Application.h`](../../Application.h.md)
**Used by** — [`AppContext.cpp`](../../AppContext.cpp.md) · [`Application.cpp`](../../Application.cpp.md) · [`audio/MumbleIntegration.cpp`](../../audio/MumbleIntegration.cpp.md) · [`audio/SoundScriptManager.cpp`](../../audio/SoundScriptManager.cpp.md) · [`gameplay/Character.cpp`](../../gameplay/Character.cpp.md) · [`gfx/EnvironmentMap.cpp`](../EnvironmentMap.cpp.md) · [`gfx/GfxScene.h`](../GfxScene.h.md) · [`gfx/GfxWater.cpp`](../GfxWater.cpp.md) · [`gfx/HydraxWater.cpp`](../HydraxWater.cpp.md) · [`gfx/ShadowManager.cpp`](../ShadowManager.cpp.md) · [`gfx/SimBuffers.h`](../SimBuffers.h.md) · [`gfx/SkyManager.cpp`](../SkyManager.cpp.md) · [`gfx/SkyXManager.cpp`](../SkyXManager.cpp.md) · [`CameraManager.cpp`](CameraManager.cpp.md) · [`gui/GUIManager.cpp`](../../gui/GUIManager.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../../gui/panels/GUI_TopMenubar.cpp.md) · [`main.cpp`](../../main.cpp.md) · [`physics/ActorSpawner.cpp`](../../physics/ActorSpawner.cpp.md) · [`physics/water/Wavefield.cpp`](../../physics/water/Wavefield.cpp.md) · [`terrain/TerrainEditor.cpp`](../../terrain/TerrainEditor.cpp.md)
**Tier floor** — T2


## Purpose

Owns the main camera and its scene node. Each frame it reads input and moves the camera according to the current behaviour. Implementation: [`CameraManager.cpp`](CameraManager.cpp.md).

## State

```text
ENUM behaviour: CHARACTER 0, STATIC 1, VEHICLE 2, VEHICLE_SPLINE 3, VEHICLE_CINECAM 4, (END 5 — cycle limit), FREE, FIXED, ISOMETRIC, INVALID −1
RECORD CameraManager
  camera ("PlayerCam", near 0.5, auto aspect), node (fixed yaw axis)
  current behaviour; behaviour before toggling (FREE/FIXED) and one-slot previous toggle
  per-frame context: player actor, dt, rotation scale, translation scale, simulation speed
  orbit: rot_x, rot_y (0.3 rad), target direction, target pitch, distance (5), min/max distance, smoothing ratio (11),
         look-at (+ last, smoothed, smoothed last), limit movement (true)
  static camera: force-update, fov exponent, previous fov, look-at, position, timer
  character camera: third person (true)
  spline camera: debug line, spline, length, position (0.5), closed, auto tracking, rail nodes, linked-actor count
```

## API

`UpdateInputEvents(dt)`, behaviour getters, camera/node access, `NotifyContextChange`, `NotifyVehicleChanged(actor?)`, orbit helpers, mouse handlers, `ResetAllBehaviors`, `ReCreateCameraNode` (after the scene is cleared), `switchToNextBehavior`, `EvaluateSwitchBehavior`.
