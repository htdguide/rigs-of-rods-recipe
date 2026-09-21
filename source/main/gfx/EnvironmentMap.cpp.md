# source/main/gfx/EnvironmentMap.cpp

> Round-robin cube-face rendering centred on the player’s vehicle, hiding the vehicle itself.

**Needs** — [`EnvironmentMap.h`](EnvironmentMap.h.md) · [`Application.h`](../Application.h.md) · [`camera/CameraManager.h`](camera/CameraManager.h.md) · [`GameContext.h`](../GameContext.h.md) · [`GfxActor.h`](GfxActor.h.md) · [`GfxScene.h`](GfxScene.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`SkyManager.h`](SkyManager.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md)
**Used by** — callers of [`EnvironmentMap.h`](EnvironmentMap.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`EnvironmentMap.h`](EnvironmentMap.h.md).

## State

See header.

## `SetupEnvMap`

Six 90° square cameras (near 0.1, far = main camera's) looking +X, −X, +Y, −Y, −Z, +Z render into the faces of the cube texture; no overlays; manual updates. With `diag_envmap` an overlay shows the unfolded cube.

## `UpdateEnvMap(centre, actor, full)`

Skipped when disabled, when a diagnostic UI hides elements, or when the per-frame rate (`gfx_envmap_rate`) is 0 (unless `full`). Move all cameras to the centre; hide the actor's meshes and rods (it must not reflect itself); render `rate` faces (6 when full) continuing from the last face rendered, notifying the sky of each camera; restore visibility. Spreading faces over frames keeps the cost bounded.
