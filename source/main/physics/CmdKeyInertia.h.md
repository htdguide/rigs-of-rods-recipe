# source/main/physics/CmdKeyInertia.h

> Named response curves ("inertia models") and the two inertia filters that shape command/hydro input and light fade.

**Needs** — nothing in this repository (only standard or third-party headers)
**Used by** — [`Actor.cpp`](Actor.cpp.md) · [`Actor.h`](Actor.h.md) · [`ActorForcesEuler.cpp`](ActorForcesEuler.cpp.md) · [`ActorManager.h`](ActorManager.h.md) · [`ActorSpawner.cpp`](ActorSpawner.cpp.md) · [`CmdKeyInertia.cpp`](CmdKeyInertia.cpp.md) · [`SimData.h`](SimData.h.md)
**Tier floor** — T2

## Purpose

Truck files can give commands, hydros, rotators and `flares3` a start/stop delay and a named curve. The curves are loaded once from `inertia_models.cfg`; each controlled element owns a small filter. Implementation: [`CmdKeyInertia.cpp`](CmdKeyInertia.cpp.md).

## State

```text
RECORD CmdKeyInertiaConfig
  splines : map<name, Spline>                  # Catmull-Rom through (x, y) points

RECORD CmdKeyInertia                           # runs in the 2 kHz physics loop
  last_output : float = 0
  start_delay, stop_delay : float = 0          # rate multipliers
  time : s = 0
  start_function, stop_function : name
  start_spline, stop_spline : Spline?          # borrowed from the config

RECORD SimpleInertia                           # runs once per rendered frame
  last_input : bool; start_delay, stop_delay : s; spline_time : 0..1
  start_spline, stop_spline : Spline?
```

## API

- `CmdKeyInertiaConfig.LoadDefaultInertiaModels()`, `GetSplineByName(name)` (none if unknown).
- `CmdKeyInertia.SetCmdKeyDelay(config, start, stop, start_fn, stop_fn)`, `CalcCmdKeyDelay(input, dt) → output`, `ResetCmdKeyDelay()`, getters.
- `SimpleInertia.SetSimpleDelay(config, start, stop, start_fn, stop_fn)`, `CalcSimpleDelay(bool input, dt) → 0..1`.
