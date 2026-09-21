# source/main/physics/air/Airfoil.h

> Lift, drag and moment coefficients over the full ±180° angle-of-attack range, loaded from an X-Plane .afl file.

**Needs** — [`Application.h`](../../Application.h.md)
**Used by** — [`physics/Actor.cpp`](../Actor.cpp.md) · [`physics/ActorForcesEuler.cpp`](../ActorForcesEuler.cpp.md) · [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`Airfoil.cpp`](Airfoil.cpp.md) · [`TurboProp.cpp`](TurboProp.cpp.md) · [`physics/flex/FlexAirfoil.cpp`](../flex/FlexAirfoil.cpp.md)
**Tier floor** — T2


## Purpose

Shared by wings ([`FlexAirfoil`](../flex/FlexAirfoil.h.md)), propeller blades ([`TurboProp`](TurboProp.cpp.md)) and fuselage drag. Implementation: [`Airfoil.cpp`](Airfoil.cpp.md).

## State

```text
RECORD Airfoil
  cl, cd, cm : array[3601] of float        # index = (angle_deg + 180)·10, 0.1° resolution
```

## API

`Airfoil(file)`, `getparams(aoa_deg, chord_ratio, deflection, out cl, out cd, out cm)`.

**Seam** — the file format belongs to X-Plane; see [Seam: Airfoil data](../../../../SYSTEM-REQUIREMENTS.md#5-data-and-persistence) in the data section.
