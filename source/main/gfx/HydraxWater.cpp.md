# source/main/gfx/HydraxWater.cpp

> Hydrax setup and per-frame update (height, sun from Caelum).

**Needs** — [`HydraxWater.h`](HydraxWater.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`AppContext.h`](../AppContext.h.md) · [`camera/CameraManager.h`](camera/CameraManager.h.md) · [`GameContext.h`](../GameContext.h.md) · [`GfxScene.h`](GfxScene.h.md) · [`SkyManager.h`](SkyManager.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md)
**Used by** — callers of [`HydraxWater.h`](HydraxWater.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`HydraxWater.h`](HydraxWater.h.md).

## State

See header.

## Behaviour

- Construction sets the camera near clip to 0.1 m and creates Hydrax with a projected-grid module on the plane y = 0 (vertex normals), loads the config, picks the shader mode by render system (HLSL for Direct3D, GLSL otherwise), creates it at the water height.
- `FrameStepWater(dt)` — follow static water-height changes, update Hydrax, then `UpdateWater`: with the Caelum sky, place the sun 80 km from the camera along the sun's light direction and use its colour.
- Visibility and sun-position setters forward to Hydrax. Destruction removes it.
