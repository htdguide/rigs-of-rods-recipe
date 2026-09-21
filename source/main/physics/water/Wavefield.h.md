# source/main/physics/water/Wavefield.h

> The water surface model: static level plus a sum of sinusoidal wave trains from wavefield.cfg.

**Needs** — [`ForwardDeclarations.h`](../../ForwardDeclarations.h.md) · [`utils/Vec3.h`](../../utils/Vec3.h.md)
**Used by** — [`Wavefield.cpp`](Wavefield.cpp.md) · [`terrain/Terrain.h`](../../terrain/Terrain.h.md)
**Tier floor** — T2


## Purpose

The physical water surface (the rendered water is a separate [seam](../../../../SYSTEM-REQUIREMENTS.md#seam-water-rendering)). One per terrain with water. Implementation: [`Wavefield.cpp`](Wavefield.cpp.md).

## State

```text
RECORD WaveTrain = { amplitude, max_height, wavelength, wavespeed = 1.25·sqrt(wavelength), direction (rad), sin, cos }
RECORD Wavefield
  trains       : list<WaveTrain>
  water_height : static level from the terrain definition
  waves_height : extra wave height (scriptable)
  max_ampl     : Σ max_height
  map_size, plane_scale (1.5 for maps smaller than 1500×1500 m, else 1)
  time         : simulation seconds (advanced per frame)
```

## API

`Get/SetStaticWaterHeight`, `SetWavesHeight`, `CalcWavesHeight(pos, timeshift)`, `CalcWavesVelocity(pos, timeshift)`, `FrameStepWaveField(dt)`, `IsUnderWater(pos)`, `GetWaveHeight(pos)`.
