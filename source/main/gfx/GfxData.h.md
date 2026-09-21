# source/main/gfx/GfxData.h

> Visual-only data: prop animation flags and modes, prop/beam/wheel/node visuals, video cameras, exhausts, particles, debug views.

**Needs** — [`utils/MeshObject.h`](../utils/MeshObject.h.md) · [`physics/SimConstants.h`](../physics/SimConstants.h.md)
**Used by** — [`GfxActor.h`](GfxActor.h.md) · [`GfxScene.h`](GfxScene.h.md) · [`physics/flex/FlexBody.h`](../physics/flex/FlexBody.h.md)
**Tier floor** — T2


## Purpose

The graphics counterpart of [`SimData.h`](../physics/SimData.h.md): records owned by [`GfxActor`](GfxActor.h.md) and [`GfxScene`](GfxScene.h.md). The prop-animation flag values are produced by the [spawner](../physics/ActorSpawner.cpp.md#graphics-plumbing) from truck-file keywords.

## State

```text
PROP_ANIM_FLAG (64-bit): AIRSPEED 1, VVI 2, ALTIMETER 3, AOA 4, FLAP 5, AIRBRAKE 6, ROLL 7, PITCH 8, THROTTLE 9, RPM 10, ACCEL 11,
  BRAKE 12, CLUTCH 13, TACHO 14, SPEEDO 15, PBRAKE 16, TURBO 17, SHIFTER 18, AETORQUE 19, AEPITCH 20, AESTATUS 21, TORQUE 22,
  HEADING 23, DIFFLOCK 24, STEERING 25, EVENT 26, AILERONS 27, ARUDDER 28, BRUDDER 29, BTHROTTLE 30, PERMANENT 31,
  ELEVATORS 32, DASHBOARD 33, SIGNALSTALK 34, GEAR 35           (bit n)
PROP_ANIM_MODE: ROTA_X 1, ROTA_Y 2, ROTA_Z 3, OFFSET_X 4, OFFSET_Y 5, OFFSET_Z 6, AUTOANIMATE 7, NOFLIP 8, BOUNCE 9
ShifterPropAnim: 1 H left/right, 2 H up/down, 3 sequential, 4 linear, 5 automatic linear
DebugViewType: NONE, SKELETON, NODES, BEAMS, WHEELS, SHOCKS, ROTATORS, SLIDENODES, SUBMESH, BUOYANCY  (cycling skips NONE)
VideoCamState: INVALID, DISABLED, ENABLED_OFFLINE, ENABLED_ONLINE

RECORD PropAnim = { ratio; flags; mode; opt3 (shifter kind / engine number / altimeter kind / dash input / gear);
                    opt5 (bounce direction); lower, upper limit; shifter smooth/step/target }
RECORD Prop = { id; ref, x, y nodes; offset (+ original); rotation (Euler° and quaternion); pivot scene node; mesh;
                media names; animations; camera visibility mode (active/original);
                steering wheel mesh, position, node, rotation degrees; beacon type ('b' custom, 'r' red, 'p' lightbar,
                'L'/'R'/'w' aircraft nav/strobe) with up to 4 billboards, nodes, lights, rotation rate (rad/s) and angle;
                aero engine index; propeller blade / spinner flags }
RECORD VideoCamera = { role; centre, dir-y, dir-z, alt-position, look-at nodes; rotation; offset; original material name;
                       material; offline texture; camera; render target / texture / window; debug node; mirror prop node }
RECORD NodeGfx = { node; wet_time (−1 = dry); no_particles; may_get_wet; is_hot; under_water_prev; no_sparks }
RECORD BeamGfx = { scene node; beam index; diameter; node1, node2 (may be retargeted, e.g. hooks); target actor; visible }
RECORD FreeBeamGfx = { id; primary free force; secondary free force?; scene node; diameter; visible }
RECORD FreeBeamGfxRequest = { id, forces (as script-friendly 64-bit ints), mesh "beam.mesh", material "tracks/beam", diameter }
RECORD WheelGfx = { wheel; flexing mesh; scene node; side; rim mesh name }
RECORD AirbrakeGfx = { mesh; scene node; entity; offset; ref, x, y nodes }
RECORD FlareMaterial = { flare index; material instance; emissive colour }
RECORD Exhaust = { emitter node; direction node; scene node; particle system?; template name }
RECORD CParticle = { emitter node; direction node; scene node; particle system }
```
