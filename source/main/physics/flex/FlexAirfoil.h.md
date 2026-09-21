# source/main/physics/flex/FlexAirfoil.h

> A wing segment: aerodynamic forces on eight nodes plus its own deforming mesh with a moving control surface.

**Needs** — [`Application.h`](../../Application.h.md) · [`SimData.h`](../SimData.h.md)
**Used by** — [`gfx/GfxActor.cpp`](../../gfx/GfxActor.cpp.md) · [`gui/OverlayWrapper.cpp`](../../gui/OverlayWrapper.cpp.md) · [`physics/Actor.cpp`](../Actor.cpp.md) · [`physics/ActorForcesEuler.cpp`](../ActorForcesEuler.cpp.md) · [`physics/ActorSpawner.cpp`](../ActorSpawner.cpp.md) · [`FlexAirfoil.cpp`](FlexAirfoil.cpp.md)
**Tier floor** — T2


## Purpose

`wings` in the truck format. Each segment is a box of 8 nodes (front/back × left/right × up/down); consecutive segments share nodes. Implementation: [`FlexAirfoil.cpp`](FlexAirfoil.cpp.md).

## State

```text
RECORD FlexAirfoil
  nodes nfld, nfrd, nflu, nfru, nbld, nbrd, nblu, nbru     # front/back, left/right, up/down
  type : control-surface letter (n none, a/b/f/e/r/c/d/g/h/i/j, S/T/U/V stabilators)
  has_control = type ∉ {n, S, T, U, V}; is_stabilator = type ∈ {S, T, U, V}; stabilator pivots on left for T/V
  chord_ratio (hinge position), min/max deflection (°), deflection
  lift_coef (the file's efficacy), airfoil, aoa (last computed, °)
  broken, breakable (local actors only), sref (squared reference area ×4)
  induced drag: enabled, span, area, left?
  wash: up to 8 (propeller index, ratio)
  mesh: 54 vertices (2·24 + 4 + 2) in 4 submeshes (faces, band, control-surface up/down) and a 30-point section profile
```

## API

`updateVerticesPhysics`, `updateVerticesGfx(gfx_actor) → centre`, `uploadVertices`, `setControlDeflection(−1..1)`, `enableInducedDrag(span, area, left)`, `addwash(prop, ratio)`, `updateForces`.
