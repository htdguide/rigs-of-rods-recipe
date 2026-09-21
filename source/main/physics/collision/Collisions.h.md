# source/main/physics/collision/Collisions.h

> The static world: collision boxes, collision triangles, a spatial hash over 2 m cells, ground models, and event sources.

**Needs** — [`Application.h`](../../Application.h.md) · [`SimData.h`](../SimData.h.md)
**Used by** — [`GameContext.cpp`](../../GameContext.cpp.md) · [`audio/SoundManager.h`](../../audio/SoundManager.h.md) · [`gameplay/Character.cpp`](../../gameplay/Character.cpp.md) · [`gameplay/Landusemap.cpp`](../../gameplay/Landusemap.cpp.md) · [`gfx/GfxActor.cpp`](../../gfx/GfxActor.cpp.md) · [`gfx/camera/CameraManager.cpp`](../../gfx/camera/CameraManager.cpp.md) · [`gui/panels/GUI_CollisionsDebug.cpp`](../../gui/panels/GUI_CollisionsDebug.cpp.md) · [`gui/panels/GUI_CollisionsDebug.h`](../../gui/panels/GUI_CollisionsDebug.h.md) · [`gui/panels/GUI_FlexbodyDebug.cpp`](../../gui/panels/GUI_FlexbodyDebug.cpp.md) · [`gui/panels/GUI_FrictionSettings.cpp`](../../gui/panels/GUI_FrictionSettings.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](../../gui/panels/GUI_SurveyMap.cpp.md) · [`main.cpp`](../../main.cpp.md) · [`physics/Actor.cpp`](../Actor.cpp.md) · [`physics/ActorForcesEuler.cpp`](../ActorForcesEuler.cpp.md) · [`physics/ActorManager.cpp`](../ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`Collisions.cpp`](Collisions.cpp.md) · [`DynamicCollisions.cpp`](DynamicCollisions.cpp.md) · [`resources/tobj_fileformat/TObjFileFormat.h`](../../resources/tobj_fileformat/TObjFileFormat.h.md) · [`scripting/GameScript.cpp`](../../scripting/GameScript.cpp.md) · [`scripting/ScriptEngine.cpp`](../../scripting/ScriptEngine.cpp.md) · [`terrain/ProceduralRoad.cpp`](../../terrain/ProceduralRoad.cpp.md) · [`terrain/Terrain.cpp`](../../terrain/Terrain.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../../terrain/TerrainObjectManager.cpp.md)
**Tier floor** — T2


## Purpose

One instance per loaded terrain. Everything that does not move — object collision boxes, mesh triangles, the heightfield — is tested here, per node, every physics step. Implementation: [`Collisions.cpp`](Collisions.cpp.md).

## State

```text
RECORD collision_box_t                    # (declared in SimData)
  virt (event-only, no physical contact), refined (globally rotated), selfrotated, camforced, enabled
  event_filter, eventsourcenum
  lo, hi         : absolute AABB (world)
  center         : rotation centre (object position)
  rot, unrot     : global rotation and inverse (refined boxes)
  selfcenter, selfrot, selfunrot           # rotation about the box's own centre
  relo, rehi     : box in object-local coordinates (scaled)
  campos         : forced camera position; reverb_preset_name; debug corner points[8]

RECORD collision_tri_t
  a, b, c : Vec3; aab (padded 0.1 m); forward, reverse : Mat3   # world ↔ triangle frame; gm : ground model; enabled

RECORD collision_mesh_t                   # diagnostics/editing only
  mesh name, source name, position, orientation, scale, bounds, ground model, first tri index, tri count, vert/index counts

RECORD eventsource_t
  instance_name (given when the object was spawned), box_name (the ODEF 'event' name),
  direction, script_handler, box index, enabled

RECORD ground_model_t                     # (declared in SimData) — read from ground_models.cfg
  va adhesion velocity, ms static friction, mc sliding friction, t2 hydrodynamic friction, vs Stribeck velocity,
  alpha, strength, fluid_density, flow_consistency_index, flow_behavior_index, solid_ground_level, drag_anisotropy,
  fx_type (NONE/HARD/DUSTY/CLUMPY/PARTICLE), fx colour, particle name and particle parameters, name, base name

RECORD Collisions
  boxes : list<collision_box_t>; tris : list<collision_tri_t>; meshes : list<collision_mesh_t>
  world_aab                                 # tight box around all static collision
  hash : array[2^20] of list<{cell_id, element}>   # element < 1 000 000 = box index, else tri index + 1 000 000
  hash_height : array[2^20] of float        # highest top of anything in that bucket
  ground_models : map<name, ground_model_t>; default_gm = "concrete", default_ground_gm = "gravel"
  eventsources : array[500]; free_eventsource
  landuse map (optional); forcecam, forcecampos; last_called_boxes (character only)
  CELL_SIZE = 2 m, MAXIMUM_CELL = 0x7FFF (terrain ≤ ~65 km per axis in cells), ground model file version = 3
```

## API

Ground models: load default / load file, get by name, land-use setup. Static geometry: add/remove box, add/remove tri, add/register mesh, finish loading. Queries: `nodeCollision`, `groundCollision`, `collisionCorrect` (character), `getSurfaceHeight(Below)`, `intersectsTris`, `intersectsTerrain`, `isInside`, `findPotentialEventBoxes`, box lookup by (instance, box) name, event callbacks, debug visualisation. Free function `primitiveCollision` — the contact law used by everything.
