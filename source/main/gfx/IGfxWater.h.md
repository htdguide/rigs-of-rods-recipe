# source/main/gfx/IGfxWater.h

> The rendered-water interface (pluggable): basic/reflective water or the Hydrax ocean.

**Needs** — [`ForwardDeclarations.h`](../ForwardDeclarations.h.md)
**Used by** — [`audio/SoundManager.cpp`](../audio/SoundManager.cpp.md) · [`GfxWater.h`](GfxWater.h.md) · [`HydraxWater.h`](HydraxWater.h.md) · [`SurveyMapTextureCreator.cpp`](SurveyMapTextureCreator.cpp.md) · [`system/ConsoleCmd.cpp`](../system/ConsoleCmd.cpp.md)
**Tier floor** — T2


## Purpose

[Seam: Water rendering](../../../SYSTEM-REQUIREMENTS.md#seam-water-rendering) is pluggable: the terrain picks an implementation from `gfx_water_mode`. Physics never uses this — it uses the [wavefield](../physics/water/Wavefield.h.md).

## State

Interface only.

## Interface

```text
INTERFACE IGfxWater
  SetWaterBottomHeight(h)            optional
  SetWaterVisible(bool)              required
  WaterSetSunPosition(pos)           optional
  FrameStepWater(dt)                 required
  SetReflectionPlaneHeight(h)        optional
  UpdateReflectionPlane(h)           optional
  WaterPrepareShutdown()             optional
  UpdateWater()                      required
  SetForcedCameraTransform(fov, pos, rot) / ClearForcedCameraTransform()   optional (used when rendering from other cameras)
```
