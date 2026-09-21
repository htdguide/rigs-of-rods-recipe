# source/main/gfx/SkyXManager.h

> SkyX sky adapter: sun/ambient lights and water colour following time of day.

**Needs** — [`Application.h`](../Application.h.md) · [Seam: Sky rendering](../../../SYSTEM-REQUIREMENTS.md#seam-sky-rendering)
**Used by** — [`GameContext.cpp`](../GameContext.cpp.md) · [`GfxScene.cpp`](GfxScene.cpp.md) · [`SkyXManager.cpp`](SkyXManager.cpp.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md)
**Tier floor** — T2


## Purpose

Wraps the vendored SkyX library (not twinned; [Seam: Sky rendering](../../../SYSTEM-REQUIREMENTS.md#seam-sky-rendering)). Implementation: [`SkyXManager.cpp`](SkyXManager.cpp.md).

## State

```text
RECORD SkyXManager = { light0 (point, sun glow), light1 (directional, main light); skyx; basic controller; config manager;
                       gradients: water, sun, ambient; last hour }
```

## API

`getMainLightDirection`, `getMainLight` (= light1), `update(dt)`, `InitLight`, `UpdateSkyLight`, memory/free stubs.
