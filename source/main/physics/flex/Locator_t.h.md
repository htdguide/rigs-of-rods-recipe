# source/main/physics/flex/Locator_t.h

> How one flexbody vertex is attached to three nodes; and the raw forvert override record.

**Needs** — [`SimData.h`](../SimData.h.md)
**Used by** — [`FlexBody.h`](FlexBody.h.md) · [`FlexFactory.h`](FlexFactory.h.md)
**Tier floor** — T2


## Purpose

Shared between [`FlexBody`](FlexBody.cpp.md) and the flexbody cache file.

## State

```text
RECORD Locator
  ref, nx, ny : node      # vertex frame: origin ref, axes ref→nx, ref→ny, and their normalised cross product
  coords      : Vec3      # vertex expressed in that (non-orthogonal) frame
  is_forvert  : bool      # chosen explicitly by 'forvert' instead of automatically
  smallest() = min(ref, nx, ny); mean() = (ref + nx + ny)/3
  biggest()  = also min(ref, nx, ny)        # a copy-paste slip; unused by the defragmenter, keep or fix freely

RECORD ForvertTempData = { nref, nx, ny : node; vert_index }   # node lookup is done by the spawner, vertex lookup by FlexBody
```

**Notes** — the locator is written raw into the flexbody cache file, so its layout is part of that (currently disabled) format.
