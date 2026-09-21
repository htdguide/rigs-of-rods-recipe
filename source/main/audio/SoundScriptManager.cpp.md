# source/main/audio/SoundScriptManager.cpp

> Parsing sound scripts, routing triggers/modulations to instances, and pitch-band crossfading.

**Needs** — [`SoundScriptManager.h`](SoundScriptManager.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`gfx/camera/CameraManager.h`](../gfx/camera/CameraManager.h.md) · [`Sound.h`](Sound.h.md) · [`SoundManager.h`](SoundManager.h.md) · [`utils/Utils.h`](../utils/Utils.h.md)
**Used by** — callers of [`SoundScriptManager.h`](SoundScriptManager.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`SoundScriptManager.h`](SoundScriptManager.h.md).

## State

See header.

## `.soundscript` format

```text
// comment lines
<template name>
{
    trigger_source <trigger>
    pitch_source   <modulation>          pitch_factors <offset> <multiplier> [<square>]
    gain_source    <modulation>          gain_factors  <offset> <multiplier> [<square>]
    start_sound    <pitch> <file>
    stop_sound     <pitch> <file>
    sound          <pitch> <file>        (up to 16; pitches strictly ascending unless a pitch is 0/"unpitched")
}
```

Tokens split on spaces/tabs. A duplicate template name is logged and its block skipped; a bad attribute line is logged. `creak` as a trigger is accepted only when `audio_enable_creak` is on (else the template has no trigger and cannot be instantiated). Non-numeric pitches parse as 0 = unpitched.

## Instances

`createInstance` — unknown template or no trigger → none; any routing table full (256) → logged, none. Registers the instance under its trigger, gain source and pitch source; `always_on` instances start immediately. `removeInstance` compacts the tables. (Its pitch-table search uses the *gain* table's count as its bound — a latent slip that only matters when the two differ.)

## Triggers (per actor, link type and item)

- `trigStart` — no-op if already on; mark on; `start()` matching instances: restart the start sound, loop all sounds.
- `trigStop` — no-op if off; mark off; `stop()`: stop loops, play the stop sound.
- `trigKill` — like stop but also cuts the start sound.
- `trigOnce` — `runOnce()`: play start sound (unless playing), each sound once (unless playing), stop sound.
- `trigToggle` — start/stop by state.

## Modulation

```text
FOR EACH instance on this gain source (matching actor/link): gain = clamp(sq·v² + mul·v + off, 0, 1)
FOR EACH instance on this pitch source: pitch = max(0, sq·v² + mul·v + off)
```

## Pitch crossfade — `setPitch(p)`

Each looped sound has a reference pitch; the two whose references bracket p play, crossfaded linearly, each resampled by `p / reference`:

```text
up = first sound with reference > p
up == 0:          sound 0 plays with fade-down gain; others silent (unpitched ones stay at 1)
up == count:      last sound at gain 1
otherwise:        low = up−1: gain (ref_up − p)/(ref_up − ref_low); up: (p − ref_low)/(ref_up − ref_low); all others silent
fade-down gain(ref, p) = 1 above ref/3, 0 below ref/5, linear between     (also for start/stop sounds)
then re-apply the last gain: each sound's gain = instance gain × its crossfade gain
```

Position and velocity go to all sounds. `update(dt)` (while simulating or editing): listener at the camera with velocity from camera motion, then the sound manager updates.
