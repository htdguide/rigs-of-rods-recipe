# source/main/gfx/particle/FireExtinguisherAffector.cpp

> Counting spray particles inside each fire’s sphere and reducing its intensity.

**Needs** — [`FireExtinguisherAffector.h`](FireExtinguisherAffector.h.md) · [`Application.h`](../../Application.h.md) · [`ExtinguishableFireAffectorFactory.h`](ExtinguishableFireAffectorFactory.h.md)
**Used by** — callers of [`FireExtinguisherAffector.h`](FireExtinguisherAffector.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`FireExtinguisherAffector.h`](FireExtinguisherAffector.h.md).

## State

See header.

## `_affectParticles(dt)`

```text
FOR EACH non-template fire
  hits = spray particles within the fire's radius of its world middle point; those particles die immediately
  IF hits > 0: remaining = fire.reduce_intensity(hits × effectiveness); IF remaining < 0: destroy the fire's particle system
```

Missing fire factory at construction is logged as an error ("Was it registered in the content manager?").
