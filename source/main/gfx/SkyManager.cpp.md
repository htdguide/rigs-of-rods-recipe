# source/main/gfx/SkyManager.cpp

> Caelum setup, fog policy, and light-map refresh as the sun moves.

**Needs** — [`SkyManager.h`](SkyManager.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`AppContext.h`](../AppContext.h.md) · [`camera/CameraManager.h`](camera/CameraManager.h.md) · [`GameContext.h`](../GameContext.h.md) · [`GfxScene.h`](GfxScene.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`terrain/TerrainGeometryManager.h`](../terrain/TerrainGeometryManager.h.md)
**Used by** — callers of [`SkyManager.h`](SkyManager.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`SkyManager.h`](SkyManager.h.md).

## State

See header.

## Behaviour

- Construction attaches Caelum (default components) to the viewport, window and frame loop.
- `LoadCaelumScript` — load the named sky; fog: if both terrain fog bounds are given and start < end, far clip = end/0.8 and linear fog start..end; otherwise (finite far clip) linear fog 0.7..0.9 of the far clip, logging if the bounds were invalid or incomplete; unlimited → no fog. The moon never lights or casts shadows; single light and single shadow source; normalise the sun direction.
- `DetectSkyUpdate` — whenever sky time advanced by more than 0.001 day (~86 s), refresh the terrain light maps.
