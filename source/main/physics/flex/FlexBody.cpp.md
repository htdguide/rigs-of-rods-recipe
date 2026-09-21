# source/main/physics/flex/FlexBody.cpp

> Binding mesh vertices to node frames at spawn, per-frame deformation, damage/wet blending, optional vertex defragmentation.

**Needs** — [`FlexBody.h`](FlexBody.h.md) · [`Application.h`](../../Application.h.md) · [`ApproxMath.h`](../ApproxMath.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`SimData.h`](../SimData.h.md) · [`FlexFactory.h`](FlexFactory.h.md) · [`gfx/GfxActor.h`](../../gfx/GfxActor.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md) · [`resources/rig_def_fileformat/RigDef_File.h`](../../resources/rig_def_fileformat/RigDef_File.h.md)
**Used by** — callers of [`FlexBody.h`](FlexBody.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

The algorithm that makes a rigid mesh bend with the beams.

## State

See [`FlexBody.h`](FlexBody.h.md).

## Construction (binding)

```text
placement: IF ref valid
             X = x − ref; Y = y − ref; N = unit(Y × X)
             position = ref + off.x·X + off.y·Y + off.z·N
             orientation = basis(unit X, N, unit X × N) · rotation
           ELSE position = node0 + offset; orientation = rotation
check the mesh: missing UVs → no texturing and no blend; missing normals → error logged
reorganise vertex buffers: position, normal, (ARGB colour if blending), (UV) in separate dynamic streams
read all vertex positions and normals (shared data first, then each non-shared submesh in order)
transform positions by (orientation, position) into the world
FOR EACH vertex p
  IF a forvert names this vertex: use its (ref, nx, ny)
  ELSE
    ref = nearest forset node to p
    nx  = nearest forset node ≠ ref
    ny  = nearest forset node ∉ {ref, nx} whose direction from ref is within 45° of perpendicular to (nx − ref)
    (missing → node 0 with an error "REF/VX/VY node not found")
  M = [nx − ref | ny − ref | unit((nx − ref) × (ny − ref))]
  coords = M⁻¹ · (p − ref); src_normal = M⁻¹ · (orientation · normal)
bounds made a cube ±max|extent| so culling survives deformation
IF console variable flexbody_defrag_enabled AND single-submesh mesh: defragment
```

## `computeFlexbody` (per frame, worker thread)

```text
IF blending: updateBlend()
centre = ref-frame placement as above (or node 0)
FOR EACH vertex
  X = nx − ref; Y = ny − ref; Z = fast_unit(X × Y)
  dst_pos    = X·c.x + Y·c.y + Z·c.z + ref − centre
  dst_normal = fast_unit(X·n.x + Y·n.y + Z·n.z)
```

The frame is *not* orthonormalised: stretching or shearing the node triangle stretches the mesh with it, which is the intended look of deformation.

## `updateBlend`

Per vertex, from its ref node's snapshot: once the node has had ground contact set alpha to 0xFF (permanent "scratched" mark); set blue to 0xFF while the node is wet, 0 otherwise. Changes mark the colour buffer dirty; `updateFlexbodyVertexBuffers` uploads positions, normals and (if dirty) colours, then moves the scene node to the centre. `reset` clears colours.

## Defragmentation (optional, console-controlled)

Greedy reordering of vertices so that consecutive vertices reference nearby node numbers (better cache locality in `computeFlexbody`): repeatedly pick the remaining vertex whose locator is "closest" to the previous one — distance = Σ over all 9 node pairs plus smallest-node and mean-node pairs of `const_penalty + prog_penalty·|a − b|` (0 when equal), penalties from console variables. Then optionally invert the permutation and rewrite UVs and indices. Purely a performance experiment; not required for correctness.
