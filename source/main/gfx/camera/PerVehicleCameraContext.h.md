# source/main/gfx/camera/PerVehicleCameraContext.h

> Remembers which camera mode each vehicle was last viewed in.

**Needs** — nothing in this repository (only standard or third-party headers)
**Used by** — [`CameraManager.cpp`](CameraManager.cpp.md) · [`physics/Actor.h`](../../physics/Actor.h.md)
**Tier floor** — T2


## Purpose

Entering a vehicle restores the camera the player last used for it.

## State

```text
RECORD PerVehicleCameraContext = { behavior : INVALID | EXTERNAL (default) | VEHICLE_3rdPERSON | VEHICLE_SPLINE | VEHICLE_CINECAM }
```
