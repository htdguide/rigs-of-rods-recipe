# source/main/gfx/particle/ExtinguishableFireAffectorFactory.h

> Creates fire affectors and keeps a list of all of them for extinguishers to find.

**Needs** — [`ExtinguishableFireAffector.h`](ExtinguishableFireAffector.h.md)
**Used by** — [`FireExtinguisherAffector.cpp`](FireExtinguisherAffector.cpp.md) · [`FireExtinguisherAffector.h`](FireExtinguisherAffector.h.md) · [`resources/ContentManager.cpp`](../../resources/ContentManager.cpp.md)
**Tier floor** — T2


## Purpose

The extinguisher needs to enumerate every live fire; the factory tracks what it created.

## State

```text
RECORD ExtinguishableFireAffectorFactory = { name "ExtinguishableFire"; affectors : list }
```

## API

`createAffector(psys)` (also records it), `getAffectorIterator()`.
