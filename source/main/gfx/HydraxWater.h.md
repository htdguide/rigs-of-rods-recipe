# source/main/gfx/HydraxWater.h

> Adapter that makes the Hydrax ocean library an IGfxWater.

**Needs** — [Seam: Water rendering](../../../SYSTEM-REQUIREMENTS.md#seam-water-rendering) · [`IGfxWater.h`](IGfxWater.h.md)
**Used by** — [`GfxActor.cpp`](GfxActor.cpp.md) · [`GfxScene.cpp`](GfxScene.cpp.md) · [`HydraxWater.cpp`](HydraxWater.cpp.md) · [`SkyXManager.cpp`](SkyXManager.cpp.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md)
**Tier floor** — T2


## Purpose

Optional high-end water (projected grid, Perlin noise) from the vendored Hydrax library (not twinned — see [Seam: Water rendering](../../../SYSTEM-REQUIREMENTS.md#seam-water-rendering)). Implementation: [`HydraxWater.cpp`](HydraxWater.cpp.md).

## State

```text
RECORD HydraxWater = { hydrax instance; wave height; water height; noise; projected-grid module; config file = "HydraxDefault.hdx" }
```
