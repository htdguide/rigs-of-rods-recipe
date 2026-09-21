# source/main/physics/air/TurboJet.h

> A turbojet engine: thrust along an axis between two nodes, spool-up dynamics, afterburner, reverser; plus its visual.

**Needs** — [`Application.h`](../../Application.h.md) · [`AeroEngine.h`](AeroEngine.h.md) · [`resources/rig_def_fileformat/RigDef_File.h`](../../resources/rig_def_fileformat/RigDef_File.h.md) · [`SimData.h`](../SimData.h.md)
**Used by** — [`gfx/GfxActor.cpp`](../../gfx/GfxActor.cpp.md) · [`physics/Actor.cpp`](../Actor.cpp.md) · [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`TurboJet.cpp`](TurboJet.cpp.md) · [`scripting/bindings/TurbojetAngelscript.cpp`](../../scripting/bindings/TurbojetAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

`turbojets` in the truck format. Physics and visuals in [`TurboJet.cpp`](TurboJet.cpp.md).

## State

```text
RECORD Turbojet : AeroEngine
  front, back, ref : node                 # thrust axis front→back; ref orients the visual
  axis : unit Vec3; ref_length            # axis length at spawn (detects breakage)
  max_dry_thrust, afterburn_thrust : kN   # from the file ('wet' thrust > 0 ⇒ afterburnable)
  reversable, reverse, afterburner_active, failed, ignition, warmup
  rpm_percent, throttle, timer, warmup_start, warmup_time = 15 s, last_flip
  radius = back_diameter/2; area = 2π(0.6·radius)²; exhaust_velocity; propwash (always 0)
  sound slots chosen by engine index 1..8

RECORD TurbojetVisual
  nozzle (scaled nozzle_length × diameter × diameter), afterburner flame (hidden by default), smoke particles, nodes
```
