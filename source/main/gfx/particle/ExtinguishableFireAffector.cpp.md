# source/main/gfx/particle/ExtinguishableFireAffector.cpp

> Fire growth, size scaling with intensity, and script notifications.

**Needs** — [`ExtinguishableFireAffector.h`](ExtinguishableFireAffector.h.md) · [`Application.h`](../../Application.h.md) · [`scripting/ScriptEngine.h`](../../scripting/ScriptEngine.h.md)
**Used by** — callers of [`ExtinguishableFireAffector.h`](ExtinguishableFireAffector.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`ExtinguishableFireAffector.h`](ExtinguishableFireAffector.h.md).

## State

See header.

## `_affectParticles(dt)`

```text
first frame: remember particle size and starting intensity
IF intensity < 0: remove all emitters (the fire is out)
ELSE
  IF not already pending and below max: intensity += growth·dt; pending
  IF pending: particle size = original size × intensity/original; notify scripts fireEvent(instance, intensity); clear pending
```

Scripts are also notified on creation and destruction.

**Notes** — the `intensity_growth` parameter is registered with the *max intensity* command, so setting it from a particle script changes the maximum instead; growth stays at its default unless set in code. Recorded as observed.
