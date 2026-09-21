# source/main/physics/water/Wavefield.cpp

> Loads wave trains and evaluates wave height and orbital velocity at a point.

**Needs** — [`Wavefield.h`](Wavefield.h.md) · [`Actor.h`](../Actor.h.md) · [`AppContext.h`](../../AppContext.h.md) · [`gfx/camera/CameraManager.h`](../../gfx/camera/CameraManager.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md) · [`utils/PlatformUtils.h`](../../utils/PlatformUtils.h.md) · [`terrain/Terrain.h`](../../terrain/Terrain.h.md)
**Used by** — callers of [`Wavefield.h`](Wavefield.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Deterministic, stateless-per-query waves so every node can ask "how high is the water here" cheaply and consistently.

## State

See [`Wavefield.h`](Wavefield.h.md).

## Loading

`wavefield.cfg` in the config directory; lines starting with `;` are comments; each other line is `wavelength, amplitude, max_height, direction_degrees` (lines with fewer than 4 numbers are ignored). Direction is converted with 57 (not 57.29…) degrees per radian — keep it for identical wave patterns.

## Wave height scale `GetWaveHeight(pos)`

`k = |pos − (map_x·scale/2, water_height, map_z·scale/2)|² / 3 000 000 + waves_height` — waves grow with distance from the map centre, so harbours near the middle stay calm.

## `CalcWavesHeight(pos, t)`

```text
IF waves disabled (gfx_water_waves off) OR multiplayer connected: RETURN water_height    # flat and identical for all peers
IF pos.y > water_height + max_ampl: RETURN water_height
T = time + t; k = GetWaveHeight(pos); h = water_height
FOR EACH train: h += min(amplitude·k, max_height) · sin(2π·(T·speed + sin_d·x + cos_d·z) / wavelength)
RETURN h
```

## `CalcWavesVelocity(pos, t)`

Zero when waves are off, in multiplayer, or above the wave band; otherwise per train: `s = 2π·amp/(wavelength/speed)`, `φ` as above; `v.y += s·cos φ`; `v.xz += (sin_d, cos_d)·s·sin φ`.

## `IsUnderWater(pos)`

With waves on and not in multiplayer: quick reject above the band, then compare with the wave height; otherwise compare with the static level. (It tests "multiplayer disabled", where height tests "connected" — the two differ while connecting.)
