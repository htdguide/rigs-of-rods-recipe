# source/main/physics/flex/FlexAirfoil.cpp

> Wing aerodynamics (lift, drag, moment, induced drag, prop wash) and the wing mesh.

**Needs** — [`FlexAirfoil.h`](FlexAirfoil.h.md) · [`air/AeroEngine.h`](../air/AeroEngine.h.md) · [`air/Airfoil.h`](../air/Airfoil.h.md) · [`ApproxMath.h`](../ApproxMath.h.md) · [`Actor.h`](../Actor.h.md) · [`SimData.h`](../SimData.h.md) · [`gfx/GfxActor.h`](../../gfx/GfxActor.h.md)
**Used by** — callers of [`FlexAirfoil.h`](FlexAirfoil.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

The flight model of a wing segment.

## State

See [`FlexAirfoil.h`](FlexAirfoil.h.md).

## `setControlDeflection(v)`

`deflection = v < 0 ? −v·min_def : v·max_def` (degrees; the actor maps aileron/elevator/rudder/flaps to v by surface letter).

## `updateVerticesPhysics`

```text
X = nfrd − nfld; ZL = nbld − nfld; ZR = nbrd − nfrd
over = |X × ZL|² > sref OR |X × ZR|² > sref            # the segment was stretched apart
broken = breakable ? (broken OR over) : over           # broken stays broken for local actors; remote ones recover
IF has_control: move the trailing profile points for the deflection (hinge at chord_ratio, angle in degrees/57)
```

## `updateForces`

```text
IF broken: RETURN
wind = −mean(v_nfld, v_nfrd) − Σ_wash 0.5·ratio·propwash(p)·axis(p)
chord_v = mean(nbld − nfld, nbrd − nfrd); chord = |chord_v|
span_v  = mean(nfrd − nfld, nbrd − nbld); span = |span_v|
lift_dir = span_v × (−wind); S = span·chord; normal = unit(chord_v × span_v)
aoa = signed angle from chord_v to (−wind projected on the chord/normal plane); sign by rotation axis vs span
(cz, cx, cm) = stabilator ? airfoil(aoa − deflection, chord_ratio, 0) : airfoil(aoa, chord_ratio, deflection)
ρ = ISA(nfld.y); w = |wind|
F = cx·0.5·ρ·w·S·wind + cz·0.5·ρ·w·chord·lift_dir               # lift_dir is not unit: its length is span·w
IF induced drag: Fi = cx²·0.25·ρ·w·area²/(π·span_total²)·wind applied to the two back nodes of the wing's tip side
M = −cm·0.5·ρ·w²·S
front four nodes += F·lift_coef·0.75/4 + normal·lift_coef·M/(4·0.25)
back four nodes  += F·lift_coef·0.25/4 − normal·lift_coef·M/(4·0.75)
```

**Notes** — lift and drag are split 75 % / 25 % front/back (quarter-chord aerodynamic centre), and the pitching moment is realised as an opposing couple between the front and back node rows.

## Mesh

A fixed 30-point normalised section profile (`refairfoilpos`: chordwise × thickness × spanwise side) is mapped into each segment's node frame; when the segment is broken, all points collapse to its root. Stabilators rotate the whole section about a pivot line on one side. Normals are per-face; the four submeshes share one vertex buffer. Bounds are a fixed ±20 m box.
