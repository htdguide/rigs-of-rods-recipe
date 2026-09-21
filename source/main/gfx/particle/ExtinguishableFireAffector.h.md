# source/main/gfx/particle/ExtinguishableFireAffector.h

> A particle affector that makes a particle system a fire with intensity that grows and can be put out (scripting builds).

**Needs** — nothing in this repository (only standard or third-party headers)
**Used by** — [`ExtinguishableFireAffector.cpp`](ExtinguishableFireAffector.cpp.md) · [`ExtinguishableFireAffectorFactory.h`](ExtinguishableFireAffectorFactory.h.md) · [`terrain/TerrainObjectManager.cpp`](../../terrain/TerrainObjectManager.cpp.md)
**Tier floor** — T2


## Purpose

Terrain objects can contain fires (particle systems with this affector). Water-spraying particle systems with the [`FireExtinguisherAffector`](FireExtinguisherAffector.h.md) reduce the intensity of fires they hit; scripts are told about intensity changes (fire-fighting gameplay). Implementation: [`ExtinguishableFireAffector.cpp`](ExtinguishableFireAffector.cpp.md).

## State

```text
RECORD ExtinguishableFireAffector (type "ExtinguishableFire")
  middle_point (local), radius = 1, intensity = 3000, max_intensity = 4000, intensity_growth = 10 /s
  original intensity and particle dimensions (first frame), update-required flag, owning particle system,
  object instance name ("unknown" until the terrain loader sets it)
```

Parameters exposed to particle scripts: `middle_point`, `intensity`, `max_intensity`, `intensity_growth`, `radius`.

## API

Getters/setters, `getAbsoluteMiddlePoint`, `isTemplate` (no parent node), `reduceIntensity(amount) → remaining`, instance name.
