# source/main/audio/Sound.cpp

> Audibility estimate and change notifications to the voice allocator.

**Needs** — [`Sound.h`](Sound.h.md) · [`SoundManager.h`](SoundManager.h.md)
**Used by** — callers of [`Sound.h`](Sound.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`Sound.h`](Sound.h.md).

## State

See header.

## Behaviour

- Every setter ignores unchanged values; otherwise it records the value and asks the manager to recompute the source with a reason (play, stop, gain, loop, pitch, position, velocity).
- `isPlaying` asks the hardware voice (false without one).
- `computeAudibility(listener)` — 0 if disabled, not meant to play (a one-shot that finished on its voice clears `should_play`), or gain 0; else by distance d: 0 beyond max distance, gain within reference distance, otherwise `gain · ref / (ref + rolloff·(d − ref))` (inverse-distance clamped).
