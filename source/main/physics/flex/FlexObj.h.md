# source/main/physics/flex/FlexObj.h

> The old-style actor body ("cab"): one dynamic mesh whose vertices are nodes.

**Needs** — [`Application.h`](../../Application.h.md) · [`SimData.h`](../SimData.h.md)
**Used by** — [`gfx/GfxActor.cpp`](../../gfx/GfxActor.cpp.md) · [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`physics/ActorSpawner.h`](../ActorSpawner.h.md) · [`FlexObj.cpp`](FlexObj.cpp.md)
**Tier floor** — T2


## Purpose

Built from `submesh`/`texcoords`/`cab` sections. At most one per actor. Implementation: [`FlexObj.cpp`](FlexObj.cpp.md).

## State

```text
RECORD CabTexcoord = { node, u, v }                    # one mesh vertex per texcoord entry
RECORD CabSubmesh  = { backmesh_type : NONE | OPAQUE | TRANSPARENT; texcoords_end; cabs_end }  # cumulative ends

RECORD FlexObj
  mesh with one submesh per CabSubmesh (material: main / "-back" / "-trans")
  vertex_nodes[vertex] : node; vertices[] : {position, normal, uv}
  indices[3·triangles] : vertex indices
  ref_area[triangle]   : 2·|cross| at spawn                    # used to hide exploded triangles
```

## API

`FlexObj(gfx_actor, nodes, texcoords, n_tri, triangles, submeshes, material, name, back_material, trans_material)`, `UpdateFlexObj() → centre`, `ScaleFlexObj(factor)`.
