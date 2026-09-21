# source/main/gfx/DustPool.cpp

> Per-kind emitter parameters derived from contact speed.

**Needs** — [`DustPool.h`](DustPool.h.md) · [`Application.h`](../Application.h.md) · [`GameContext.h`](../GameContext.h.md) · [`GfxScene.h`](GfxScene.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`GfxWater.h`](GfxWater.h.md)
**Used by** — callers of [`DustPool.h`](DustPool.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`DustPool.h`](DustPool.h.md).

## State

See header.

## `update`

```text
FOR EACH requested slot i (speed v = |vel|, direction d)
  time factor follows simulation speed; enable emitter
  unless ripple: place at the position, direction d, particle speed v
  NORMAL (dust):  horizontal half-direction; alpha 0.05·v; lifetime 0.5·v
  CLUMP:          half direction, never downward; alpha 1
  RUBBER (smoke): horizontal quarter-direction; grey 0.9; alpha 0.1·√v; lifetime 0.25·v
  SPARKS:         template defaults
  VAPOUR:         speed v/2; grey; alpha 0.03·rate; lifetime 0.3·rate      (rate = 5 − wet time)
  DRIP:           emission rate = rate
  SPLASH:         upward (flip y, halve); grey; alpha 0.04·√v; lifetime 0.25·v
  RIPPLE:         at the static water level − 2 cm; grey; alpha 0.04·v; lifetime 0.4·v
disable all unrequested slots; allocated = 0
```

Dust and clump colours come from the ground model's fx colour.
