# source/main/gfx/particle/FireExtinguisherAffector.h

> A particle affector for water/foam sprays that consumes its particles on contact with fires.

**Needs** — [`ExtinguishableFireAffectorFactory.h`](ExtinguishableFireAffectorFactory.h.md)
**Used by** — [`FireExtinguisherAffector.cpp`](FireExtinguisherAffector.cpp.md) · [`FireExtinguisherAffectorFactory.h`](FireExtinguisherAffectorFactory.h.md)
**Tier floor** — T2


## Purpose

Implementation: [`FireExtinguisherAffector.cpp`](FireExtinguisherAffector.cpp.md).

## State

```text
RECORD FireExtinguisherAffector (type "FireExtinguisher") = { effectiveness = 1 (1 = water); fire factory (looked up by name) }
```
