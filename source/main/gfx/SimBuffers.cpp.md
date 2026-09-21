# source/main/gfx/SimBuffers.cpp

> Out-of-line constructor/destructor for the game-context buffer.

**Needs** — [`SimBuffers.h`](SimBuffers.h.md) · [`physics/Actor.h`](../physics/Actor.h.md)
**Used by** — callers of [`SimBuffers.h`](SimBuffers.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Exists only because the buffer holds an actor handle whose full type must be visible where the record is built and destroyed. A rebuild needs nothing here.

## State

Stateless.
