# source/main/gfx/SkyXManager.cpp

> Gradient-driven light and water colours, sunrise/sunset light switching.

**Needs** — [`SkyXManager.h`](SkyXManager.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`AppContext.h`](../AppContext.h.md) · [`camera/CameraManager.h`](camera/CameraManager.h.md) · [`GameContext.h`](../GameContext.h.md) · [`GfxScene.h`](GfxScene.h.md) · [`HydraxWater.h`](HydraxWater.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`terrain/TerrainGeometryManager.h`](../terrain/TerrainGeometryManager.h.md)
**Used by** — callers of [`SkyXManager.h`](SkyXManager.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`SkyXManager.h`](SkyXManager.h.md).

## State

See header.

## Behaviour

- Setup: ambient 0.35 grey; gradients keyed by sun height `p = (sun_dir.y + 1)/2`: water colour blue-teal fading to near-black (×0.4 at 1 … ×0.025 at 0), sun colour warm to grey, ambient white 1 → 0.05; load the `.skx` config and create.
- `UpdateSkyLight` each frame: sun position = camera − light_dir × skydome radius; Hydrax water colour and sun position (×0.1) when Hydrax is active; light0 at sun×0.02; light1 direction = light_dir, diffuse = ambient gradient; built-in water sun position; light0 visible only between sunrise and sunset (time vs sunrise/sunset from the controller); refresh terrain light maps once per game hour.
