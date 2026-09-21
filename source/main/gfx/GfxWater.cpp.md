# source/main/gfx/GfxWater.cpp

> Water modes, plane placement, reflection/refraction scheduling, and vertex waves.

**Needs** — [`GfxWater.h`](GfxWater.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`AppContext.h`](../AppContext.h.md) · [`camera/CameraManager.h`](camera/CameraManager.h.md) · [`GfxScene.h`](GfxScene.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md)
**Used by** — callers of [`GfxWater.h`](GfxWater.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GfxWater.h`](GfxWater.h.md).

## State

See header.

## Modes (`gfx_water_mode`)

- BASIC — plane with material `tracks/basicwater`.
- REFLECT — Fresnel reflection only (reflection texture, reflection camera mirrored about the water plane with a clip plane at water + 0.15 m).
- FULL_FAST / FULL_HQ — Fresnel reflection + refraction (refraction camera clipped below water − 0.15 m). Requires vertex and fragment programs (arbfp1, ps_2_0 or ps_1_4), else an error "Your card does not support … Water effects".
- A sea-bottom plane (`tracks/seabottom`) at the bottom height; both planes cover the (scaled) map size.

## `UpdateWater` (per frame, after `FrameStepWater` picks up height changes)

```text
camera = forced transform or the main camera
sight point = where the view ray meets the water plane (ahead of the camera), else below the camera
plane centre = camera-above-water point moved toward the sight point by min(distance, half the map's smaller side)
move the water and bottom planes there only when > 200 m away (or forced) — avoids swimming texture
IF waves enabled and single-player: set each plane vertex height from the wavefield, recompute normals from neighbours, upload
render targets: FULL_FAST alternates reflection (odd frames) and refraction (even frames); FULL_HQ both each frame; REFLECT reflection
```

During reflection/refraction renders, shadows in the main queue and the water plane itself are hidden (refraction also hides particles so spray does not refract).

## `UpdateReflectionPlane(h)`

Water plane at h; camera under water → disable reflection; else reflect about h with clip planes h ± 0.15.
