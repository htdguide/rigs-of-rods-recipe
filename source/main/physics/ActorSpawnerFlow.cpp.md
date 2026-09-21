# source/main/physics/ActorSpawnerFlow.cpp

> The fixed processing order of truck-file elements — the dependency order of the actor build.

**Needs** — [`ActorSpawner.h`](ActorSpawner.h.md) · [`Actor.h`](Actor.h.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md)
**Used by** — callers of [`ActorSpawner.h`](ActorSpawner.h.md) — it implements the spawn sequence
**Tier floor** — T2

## Purpose

Separated from the element processors only to keep the order readable. The order is load-bearing: elements may only refer to things created earlier (a beam refers to nodes; a rail to beams; a slide node to rails; props to materials; wings to the graphics actor).

## State

Stateless (drives the spawner's state).

## `ProcessNewActor(actor, request, document)`

**Contract** — builds the actor in place. Each element list is processed for every selected module in module order (root first), recording the current keyword and module for messages; an exception in one entry is reported and that entry skipped.

```text
create scene-graph grouping nodes (names must be globally unique: include the instance id)
reset wing/fuselage accumulators; disable wing position lights if any selected module has an 'engine'
InitializeRig()                                    # allocate exact-size arrays, set defaults
prepare the built-in 'renderdash' material and texture-dashboard render layers   # before any props
copy name, file hash, flags (forwardcommands, importcommands, rescuer, disabledefaultsounds, hideInChooser)
PROCESS minimass, set_collision_range, author
warn "vehicle uses no GUID, skinning will be impossible" if the root module has no guid
PROCESS description, managedmaterials (must precede any mesh), globals, help (must precede guisettings),
        engine, engoption, engturbo, torquecurve, brakes, customdashboardinputs, guisettings, scripts
create the graphics actor
PROCESS nodes
IF exhaust point and direction node flags both set: add the legacy exhaust
PROCESS cinecam                                    # generates nodes
PROCESS wheels, wheels2, meshwheels, meshwheels2, flexbodywheels   # generate nodes
PROCESS wheeldetachers
PROCESS beams, shocks, shocks2, shocks3, commands2, hydros, triggers, ropes
PROCESS antilockbrakes, flares2, flares3, flaregroups_no_import, axles, transfercase, interaxles,
        submesh, contacters, cameras, hooks, ties, ropables, animators, fusedrag, turbojets, props,
        tractioncontrol, rotators, rotators2, lockgroups, railgroups, slidenodes, particles,
        cruisecontrol, speedlimiter, collisionboxes, exhausts, extcamera, camerarail,
        pistonprops, turboprops2, screwprops, fixes,
        flexbodies, wings, airbrakes                # these three need the graphics actor
PROCESS soundsources, soundsources2 (when audio is compiled in)
FinalizeRig()
IF cab texcoords and cab triangles exist: CreateCabVisual()
FinalizeGfxSetup()
```

**Notes** — the node numbering a truck file sees is exactly this creation order: user nodes, then cinecam nodes, then wheel-generated nodes in the order wheels → wheels2 → meshwheels → meshwheels2 → flexbodywheels. Legacy `commands` and `turboprops` arrive already converted to `commands2`/`turboprops2` by the parser.
