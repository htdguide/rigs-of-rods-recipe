# source/main/physics/Differentials.h

> Differential and transfer-case records: how drive torque is shared between two wheels or two axles.

**Needs** — nothing in this repository (only standard or third-party headers)
**Used by** — [`gfx/GfxActor.h`](../gfx/GfxActor.h.md) · [`gfx/SimBuffers.h`](../gfx/SimBuffers.h.md) · [`Actor.cpp`](Actor.cpp.md) · [`Actor.h`](Actor.h.md) · [`ActorForcesEuler.cpp`](ActorForcesEuler.cpp.md) · [`ActorSpawner.cpp`](ActorSpawner.cpp.md) · [`Differentials.cpp`](Differentials.cpp.md)
**Tier floor** — T2

## Purpose

Data for the drivetrain split used by the wheel step in [`ActorForcesEuler.cpp`](ActorForcesEuler.cpp.md). Laws in [`Differentials.cpp`](Differentials.cpp.md).

## State

```text
RECORD DifferentialData          # in/out parameter block for one evaluation
  speed[2]        : rad/s-ish wheel (or axle) speeds
  delta_rotation  : accumulated relative rotation of side 0 vs side 1 (persisted by the caller)
  in_torque       : torque entering the differential
  out_torque[2]   : result
  dt              : step length

RECORD TransferCase
  axle_1 : int                   # always driven
  axle_2 : int                   # driven only in 4WD (−1 = none)
  has_2wd, has_2wd_lo : bool
  four_wd_mode : bool = false
  gear_ratios  : list<float>     # reduction ratios; the first is the active one (rotated by the "shift" action)

ENUM DiffType = SPLIT | OPEN | VISCOUS | LOCKED | INVALID

RECORD Differential
  idx_1, idx_2    : int          # wheels (wheel differential) or axles (axle differential)
  delta_rotation  : float = 0
  available       : list<DiffType>   # the first is active
```

## `Differential` API

- `AddDifferentialType(t)` — append a selectable mode.
- `ToggleDifferentialMode()` — rotate the list left by one (no-op with a single mode).
- `CalcAxleTorque(data)` — apply the active mode's law; no modes → untouched.
- `GetActiveDiffType`, `GetNumDiffTypes`, `GetDifferentialTypeName` ("Split", "Open", "Viscous", "Locked", "invalid"; localised).
