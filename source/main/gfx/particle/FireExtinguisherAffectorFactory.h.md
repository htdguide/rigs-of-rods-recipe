# source/main/gfx/particle/FireExtinguisherAffectorFactory.h

> Registers the "FireExtinguisher" affector type.

**Needs** — [`FireExtinguisherAffector.h`](FireExtinguisherAffector.h.md)
**Used by** — [`resources/ContentManager.cpp`](../../resources/ContentManager.cpp.md)
**Tier floor** — T2


## Purpose

Plugs the extinguisher affector into the particle system under its type name.

## State

Stateless (keeps created affectors in a list it never reads).

## `createAffector(psys)`

New [`FireExtinguisherAffector`](FireExtinguisherAffector.h.md).
