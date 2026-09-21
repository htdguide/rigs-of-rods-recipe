# source/main/physics/ActorSpawner.h

> The builder that turns a parsed truck document (plus chosen modules and add-on parts) into a live actor.

**Needs** — [`Application.h`](../Application.h.md) · [`resources/rig_def_fileformat/RigDef_Parser.h`](../resources/rig_def_fileformat/RigDef_Parser.h.md) · [`gui/RTTLayer.h`](../gui/RTTLayer.h.md) · [`SimData.h`](SimData.h.md) · [`flex/FlexFactory.h`](flex/FlexFactory.h.md) · [`flex/FlexObj.h`](flex/FlexObj.h.md)
**Used by** — [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`Actor.cpp`](Actor.cpp.md) · [`ActorManager.cpp`](ActorManager.cpp.md) · [`ActorSpawner.cpp`](ActorSpawner.cpp.md) · [`ActorSpawnerFlow.cpp`](ActorSpawnerFlow.cpp.md) · [`physics/flex/FlexFactory.cpp`](flex/FlexFactory.cpp.md)
**Tier floor** — T2

## Purpose

A single-use builder object: one spawner builds one actor, then is discarded. The class declares one `Process<Element>` function per truck-file element type plus the low-level node/beam allocators and graphics helpers. The element processors are in [`ActorSpawner.cpp`](ActorSpawner.cpp.md); the processing *order* is in [`ActorSpawnerFlow.cpp`](ActorSpawnerFlow.cpp.md). `FlexFactory` is a friend because it needs the spawner's naming and entity-setup helpers.

## State

```text
RECORD ActorSpawner
  actor, document
  selected_modules    : list<Module>     # root, then the chosen section-config module, then add-on-part modules
  spawn_position      : Vec3
  custom_resource_group                  # where per-actor clones of materials live (the bundle's group)
  memory_requirements : {nodes, beams, shocks, rotators, wings, airbrakes, fixes}   # pre-counted, see CalcMemoryRequirements
  global_minimass     : kg = 50          # from 'minimass'
  named_nodes         : map<name, index>
  current_keyword, current_module        # for error messages and add-on media lookup
  wing bookkeeping    : first_wing_index, wing_area, left/right position-light nodes, generate_wing_position_lights
  fuselage extent     : fuse_z_min/max, fuse_y_min/max (from node positions, for fusedrag autocalc)
  old-style cab data  : texcoords[], submeshes[] (cab triangle ranges and backmesh type)
  material_substitutions : map<original name, CustomMaterial>   # exactly one per-actor substitute per original
  managed_materials   : map<name, material>
  dashboard render-to-texture layers, mirror-prop tracking, scene-graph grouping nodes
```

`CustomMaterial` = { material, optional material-flare binding, optional video-camera definition, mirror-prop side (none/left/right) and its scene node }.

## API

- `ConfigureSections(sectionconfig, document)` — select the root module and, if named, the user module; unknown name → warning.
- `ConfigureAddonParts(actor)` — for each add-on part in the actor's working tuneup, load it and convert it into an extra module; missing/unloadable parts → warning.
- `ConfigureAssetPacks(actor)` — load every asset pack any selected module names, and record the used ones.
- `ProcessNewActor(actor, request, document)` — build everything (see [`ActorSpawnerFlow.cpp`](ActorSpawnerFlow.cpp.md)).
- `SetupDefaultSoundSources(actor)` — static; attach default sound scripts according to the actor's features.
- `GetMemoryRequirements()`, `GetSubmeshGroundmodelName()` (first module that names one), `GetActor()`.

Errors inside one element are caught per element: the element is skipped with a console message and processing continues.
