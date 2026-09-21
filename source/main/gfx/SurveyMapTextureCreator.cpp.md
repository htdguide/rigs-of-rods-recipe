# source/main/gfx/SurveyMapTextureCreator.cpp

> Orthographic top-down render with water updated from the map camera.

**Needs** — [`SurveyMapTextureCreator.h`](SurveyMapTextureCreator.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`Application.h`](../Application.h.md) · [`GameContext.h`](../GameContext.h.md) · [`GfxScene.h`](GfxScene.h.md) · [`IGfxWater.h`](IGfxWater.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md)
**Used by** — callers of [`SurveyMapTextureCreator.h`](SurveyMapTextureCreator.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`SurveyMapTextureCreator.h`](SurveyMapTextureCreator.h.md).

## State

See header.

## Behaviour

`init` creates an RGB render texture and an orthographic camera looking straight down (no overlays, shadows or sky, black background). `update(centre, size)` sets the ortho window and places the camera above the centre, then renders once; before rendering, water is updated as if seen from the map camera (forced camera transform), after rendering it is updated again for the real camera. `convertTextureToStatic` copies the render into a regular texture so the render target can be released.
