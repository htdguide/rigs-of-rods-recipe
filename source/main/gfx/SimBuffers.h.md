# source/main/gfx/SimBuffers.h

> Per-frame snapshots of simulation state that the renderer reads while physics keeps running.

**Needs** — [`gameplay/AutoPilot.h`](../gameplay/AutoPilot.h.md) · [`camera/CameraManager.h`](camera/CameraManager.h.md) · [`physics/Differentials.h`](../physics/Differentials.h.md) · [`physics/SimData.h`](../physics/SimData.h.md)
**Used by** — [`GfxActor.h`](GfxActor.h.md) · [`GfxData.cpp`](GfxData.cpp.md) · [`GfxScene.h`](GfxScene.h.md) · [`SimBuffers.cpp`](SimBuffers.cpp.md)
**Tier floor** — T2


## Purpose

The decoupling contract between simulation and graphics. Physics runs on its own thread at 2 kHz; at a well-defined point each frame (while the simulation thread is synchronised) everything the visuals need is copied into these buffers, the simulation resumes, and the scene is updated from the copies — so rendering never reads half-updated physics and physics never waits for rendering work like flexbody deformation.

Triplets {simulation object / buffer / visual}: game context / `GameContextSB` / GfxScene; actor / `ActorSB` / GfxActor; node / `NodeSB` / NodeGfx; command key / `CommandKeySB`; aero engine / `AeroEngineSB` / turbojet visual; airbrake / `AirbrakeSB` / AirbrakeGfx; screw prop / `ScrewpropSB`.

## State

```text
RECORD NodeSB = { position; has_contact : bit; is_wet : bit }
RECORD ScrewpropSB = { rudder; throttle }      RECORD CommandKeySB = { value }      RECORD PropAnimKeySB = { active }
RECORD AeroEngineSB = { type; rpm; rpm %; throttle; turboprop torque; turboprop pitch; afterburner thrust; exhaust velocity;
                        ignition; failed; afterburner on }
RECORD AirbrakeSB = { ratio }
RECORD ActorSB
  state, physics paused, current cinecam, net username, net colour, driveable type
  position, node-0 velocity, heading, direction, wheel speed, top speed, bounding box, camera-0 position/roll nodes
  nodes[], screw props[], command keys[0..84], prop animation keys[], aero engines[], airbrakes[]
  steering hydro state, brake, has engine, gear, autoshift, rpm, crank factor, turbo psi, throttle, clutch torque,
  input shaft rpm, drive ratio, active differential type, clutch, gear count, max rpm, smoke
  tyre pressure + pressurising, light mask, smoke enabled, parking brake, custom particles
  aileron / elevator / aero rudder hydro states, flap, airbrake, wing-4 angle of attack
  autopilot: present, heading mode/value, altitude mode/value, IAS mode/value, GPWS, ILS available + deviations, VS
  gui settings: speedo max km/h, use engine max rpm, shifter animation time
RECORD GameContextSB
  player actor, character position, paused, simulation speed, camera behaviour,
  race: time, best, diff, in progress (+ previous), direction arrow target/text/visible
```

Defaults mirror the owning objects' resets (e.g. autopilot altitude 1000, IAS 150).
