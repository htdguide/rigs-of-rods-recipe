# source/main/utils/MeshObject.cpp

> Mesh loading with manual and generated LODs keyed to the sight-range setting.

**Needs** — [`MeshObject.h`](MeshObject.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine)
**Used by** — callers of [`MeshObject.h`](MeshObject.h.md) (see its Used by)
**Tier floor** — T2

## Purpose

Content authors can ship hand-made LOD meshes beside a mesh by naming convention; this file discovers them. The convention is part of the content format.

## State

See header twin.

## LOD discovery (first load of a mesh only)

**Contract** — LODs must be registered before the first entity of a mesh exists, so this runs only when the mesh was not yet loaded.

```text
FUNCTION load_with_lods(mesh_name, group)
  mesh = load mesh_name from group
  base = mesh_name without extension
  any_lod = false
  # Classic LODs: "<base>_lod<N>.mesh", N = 1..4, distance from sight range
  FOR EACH file MATCHING base + "_lod*.mesh" IN group
    N = integer after "_lod"; skip if not parsable or N < 0
    IF gfx_sight_range > UNLIMITED_SIGHTRANGE           # unlimited sight
      distance = {1: 200, 2: 600, 3: 2000, 4: 5000}[N] else 3
    ELSE
      distance = max(20, sight_range * {1: 0.1, 2: 0.2, 3: 0.3, 4: 0.4}[N]) else 3
    add manual LOD level (distance, file); any_lod = true
  # Custom LODs: "<base>_clod_<D>.mesh", D = the switch distance itself
  FOR EACH file MATCHING base + "_clod_*.mesh" IN group
    D = integer after "_clod_"; skip if not parsable or D < 0
    add manual LOD level (D, file); any_lod = true
  IF any_lod: build the LOD chain
  ELSE IF gfx_auto_lod: generate LODs automatically (engine's mesh simplifier)
```

Then create entity `entity_name`, set shadow casting, attach to the node, make the node visible.

**Notes** — `UNLIMITED_SIGHTRANGE` is a threshold defined by [`Terrain`](../terrain/Terrain.h.md); sight ranges above it mean "no far clip". A LOD index outside 1–4 gets distance 3 m, which effectively means "switch almost immediately" — an original quirk; a rebuild may treat such files as errors.
