# source/main/physics/Differentials.cpp

> The four torque-split laws.

**Needs** — [`Application.h`](../Application.h.md) · [`Differentials.h`](Differentials.h.md) · [`utils/Language.h`](../utils/Language.h.md)
**Used by** — callers of [`Differentials.h`](Differentials.h.md) (see its Used by)
**Tier floor** — T2

## Purpose

Pure functions over a [`DifferentialData`](Differentials.h.md) block, evaluated once per physics step per differential.

## State

Stateless (the caller persists `delta_rotation`).

## Laws

```text
SPLIT:   out0 = out1 = in/2

OPEN:    total = |s0| + |s1|
         ratio = |s0| / total   IF min(|s0|,|s1|) > 1   ELSE 0.5
         out0 = in · clamp(ratio,     0.1, 0.9)
         out1 = in · clamp(1 − ratio, 0.1, 0.9)            # more torque to the faster side

VISCOUS: Δ = s0 − s1
         out0 = in/2 − 10000·Δ
         out1 = in/2 + 10000·Δ                             # couples speeds; more torque to the slower side

LOCKED:  Δ = s0 − s1; delta_rotation += Δ·dt
         k = 1 000 000; c = k/100
         out0 = in/2 − delta_rotation·k − Δ·c
         out1 = in/2 + delta_rotation·k + Δ·c               # torsion spring+damper keeps relative angle
```

**Notes** — "open" multiplies the *full* input torque by the ratio on each side (not half), so the two outputs sum to the input only when both ratios lie inside the clamp. The original says "RoR needs to model reaction torque" for a true open differential; this is its approximation and must be kept for identical driving feel.
