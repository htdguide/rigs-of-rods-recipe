# source/main/physics/water/ScrewProp.h

> A boat propeller with rudder: a thrust force at one node.

**Needs** — [`Application.h`](../../Application.h.md) · [`utils/memory/RefCountingObject.h`](../../utils/memory/RefCountingObject.h.md) · [`utils/memory/RefCountingObjectPtr.h`](../../utils/memory/RefCountingObjectPtr.h.md) · [`SimData.h`](../SimData.h.md)
**Used by** — [`GameContext.cpp`](../../GameContext.cpp.md) · [`audio/SoundManager.cpp`](../../audio/SoundManager.cpp.md) · [`gameplay/VehicleAI.cpp`](../../gameplay/VehicleAI.cpp.md) · [`gui/OverlayWrapper.cpp`](../../gui/OverlayWrapper.cpp.md) · [`physics/Actor.cpp`](../Actor.cpp.md) · [`physics/Actor.h`](../Actor.h.md) · [`physics/ActorForcesEuler.cpp`](../ActorForcesEuler.cpp.md) · [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`physics/Savegame.cpp`](../Savegame.cpp.md) · [`ScrewProp.cpp`](ScrewProp.cpp.md) · [`scripting/bindings/ScrewpropAngelscript.cpp`](../../scripting/bindings/ScrewpropAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

`screwprops` in the truck format. Reference-counted (scriptable). Implementation: [`ScrewProp.cpp`](ScrewProp.cpp.md).

## State

```text
RECORD Screwprop
  ref, back, up : node         # thrust at ref, direction ref→back, rudder axis ref−up
  full_power    : "HP" (used directly as newtons of thrust)
  throttle      : 0..1; reverse : bool; rudder : degrees (−45..45)
  splash, ripple particle pools
```

## API

`updateForces(update)`, `setThrottle(−1..1)`, `setRudder(−1..1)`, `getThrottle` (negative when reversing), `getRudder`, `getMaxPower`, `getReverse`, `reset`, `toggleReverse`, node getters.
