# source/main/gameplay/AutoPilot.h

> Aircraft autopilot state: heading (wings-level, fixed, NAV/ILS), altitude (fixed, vertical speed), airspeed hold, GPWS.

**Needs** — [`Application.h`](../Application.h.md) · [`utils/memory/RefCountingObject.h`](../utils/memory/RefCountingObject.h.md) · [`utils/memory/RefCountingObjectPtr.h`](../utils/memory/RefCountingObjectPtr.h.md) · [`terrain/TerrainObjectManager.h`](../terrain/TerrainObjectManager.h.md)
**Used by** — [`AutoPilot.cpp`](AutoPilot.cpp.md) · [`gfx/GfxActor.h`](../gfx/GfxActor.h.md) · [`gfx/SimBuffers.h`](../gfx/SimBuffers.h.md) · [`gui/OverlayWrapper.cpp`](../gui/OverlayWrapper.cpp.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`scripting/bindings/AutopilotAngelscript.cpp`](../scripting/bindings/AutopilotAngelscript.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md)
**Tier floor** — T2


## Purpose

Created for local aircraft at spawn (first aero engine). The actor asks it for aileron/elevator/throttle contributions each frame; the aircraft panel and scripts adjust its settings. `force_disabled` lets a script autopilot use the settings while the built-in controller stays silent. Implementation: [`AutoPilot.cpp`](AutoPilot.cpp.md).

## State

```text
RECORD Autopilot
  heading_mode : NONE | FIXED | WLV (wings level) | NAV;  heading : 0..359°
  alt_mode : NONE | FIXED | VS;  alt : ft = 1000;  vs : ft/min = 0 (±9900)
  ias_mode : bool; ias : kt = 150 (0..350); gpws : bool = true; force_disabled; wants_disconnect
  references: left wingtip, right wingtip, tail (fuselage back), cockpit (camera node 0); span = |left − right|
  filtered last aileron/elevator/rudder; last GPWS height; ILS: vertical/horizontal available,
  deviations (°, −90 = none), runway heading, runway distance, last closest horizontal distance
```

## API

`reset`, `disconnect` (all modes off; "AP disconnect" callout if GPWS on), `setForceDisabled`, `setInertialReferences`, toggles (heading/alt with mode argument: same mode again turns it off; IAS; GPWS), adjusters (HDG wraps 0..359, ALT, VS clamp, IAS clamp), outputs (`getAilerons`, `getElevator`, `getRudder` = 0, `getThrottle(pilot, dt)`), `gpws_update(spawn height)`, `UpdateIls`, getters.
