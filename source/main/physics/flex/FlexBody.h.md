# source/main/physics/flex/FlexBody.h

> A mesh skinned to the soft body: every vertex follows a frame of three nearby nodes.

**Needs** — [`resources/rig_def_fileformat/RigDef_Prerequisites.h`](../../resources/rig_def_fileformat/RigDef_Prerequisites.h.md) · [`Application.h`](../../Application.h.md) · [`Locator_t.h`](Locator_t.h.md) · [`SimData.h`](../SimData.h.md) · [`gfx/GfxData.h`](../../gfx/GfxData.h.md) · [`resources/rig_def_fileformat/RigDef_File.h`](../../resources/rig_def_fileformat/RigDef_File.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md)
**Used by** — [`gfx/GfxActor.cpp`](../../gfx/GfxActor.cpp.md) · [`gui/panels/GUI_FlexbodyDebug.cpp`](../../gui/panels/GUI_FlexbodyDebug.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../../gui/panels/GUI_TopMenubar.cpp.md) · [`physics/Actor.cpp`](../Actor.cpp.md) · [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`FlexBody.cpp`](FlexBody.cpp.md) · [`FlexFactory.cpp`](FlexFactory.cpp.md)
**Tier floor** — T2


## Purpose

`flexbodies` (and the tyre of `flexbodywheels`) — the main way vehicles look detailed while deforming. Owned by the graphics actor; deformation runs on worker threads from the node snapshot. Implementation: [`FlexBody.cpp`](FlexBody.cpp.md).

## State

```text
RECORD FlexBody
  id, placeholder_type : NOT | TUNING_REMOVED | FAULTY_FORSET | FAULTY_MESH   # placeholders keep ids stable
  ref, x, y : node; centre_offset                                            # placement frame
  vertex_count; locators[vertex] : Locator                                   # see Locator_t
  src_normals[vertex] (in locator frames), dst_pos[vertex], dst_normals[vertex]
  src_colors[vertex] : ARGB              # "texture blend": A = damaged (touched ground), B = wet
  flags: uses_shared_vertex_data, has_texture, has_texture_blend, blend_changed
  per-submesh vertex buffers (≤ 16) and shared buffers
  camera visibility mode (orig/active), forset nodes, mesh info strings, scene node + entity
```

## API

`computeFlexbody` (worker), `updateFlexbodyVertexBuffers` (main), `reset`, `updateBlend`, `writeBlend`, visibility and shadows, vertex locator/position accessors, `destroyOgreObjects`, placeholder constructor and type names.
