# source/main/physics/air/AeroEngine.h

> The common interface of aircraft engines (turbojets and propellers).

**Needs** — [`Application.h`](../../Application.h.md) · [`utils/memory/RefCountingObject.h`](../../utils/memory/RefCountingObject.h.md) · [`utils/memory/RefCountingObjectPtr.h`](../../utils/memory/RefCountingObjectPtr.h.md) · [`SimData.h`](../SimData.h.md)
**Used by** — [`GameContext.cpp`](../../GameContext.cpp.md) · [`audio/SoundManager.cpp`](../../audio/SoundManager.cpp.md) · [`gameplay/VehicleAI.cpp`](../../gameplay/VehicleAI.cpp.md) · [`gui/OverlayWrapper.cpp`](../../gui/OverlayWrapper.cpp.md) · [`physics/Actor.h`](../Actor.h.md) · [`physics/ActorForcesEuler.cpp`](../ActorForcesEuler.cpp.md) · [`physics/Savegame.cpp`](../Savegame.cpp.md) · [`TurboJet.h`](TurboJet.h.md) · [`TurboProp.h`](TurboProp.h.md) · [`physics/flex/FlexAirfoil.cpp`](../flex/FlexAirfoil.cpp.md) · [`scripting/bindings/AircraftEngineAngelscript.cpp`](../../scripting/bindings/AircraftEngineAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

Actors hold up to 8 aero engines behind one interface so the step, the dashboard, sound, savegames and the network code treat jets and props alike. Engines are reference-counted because scripts can hold them.

## State

Interface only.

## Interface

```text
INTERFACE AeroEngine
  updateForces(dt, do_update)             # per physics step; do_update = first step of the frame (sounds, density)
  setThrottle(0..1), getThrottle
  reset, flipStart (toggle ignition, debounced 0.3 s), toggleReverse, setReverse, getReverse
  getRPM, getRPMpc (percent), setRPM, getpropwash (m/s), getAxis (unit, thrust direction)
  isFailed, getType (TURBOJET | XPROP), getIgnition, setIgnition, getWarmup, getRadius
  getNoderef (reference node for sound/position), GetFrontNode, GetBackNode
  updateVisuals(graphics actor), setVisible
```
