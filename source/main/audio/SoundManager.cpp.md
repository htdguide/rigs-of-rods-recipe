# source/main/audio/SoundManager.cpp

> Voice allocation by audibility, WAV parsing, and the EFX environment model (reverb zones, obstruction, occlusion, early reflections, directed sources, underwater).

**Needs** — [`SoundManager.h`](SoundManager.h.md) · [`physics/air/AeroEngine.h`](../physics/air/AeroEngine.h.md) · [`Application.h`](../Application.h.md) · [`gfx/IGfxWater.h`](../gfx/IGfxWater.h.md) · [`physics/water/ScrewProp.h`](../physics/water/ScrewProp.h.md) · [`Sound.h`](Sound.h.md)
**Used by** — callers of [`SoundManager.h`](SoundManager.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`SoundManager.h`](SoundManager.h.md).

## State

See header.

## Startup

Open the configured device (fallback to default, clearing the setting); no device → sound disabled (everything becomes a no-op). Create the context. If the EFX extension exists and `audio_enable_efx`: pick the reverb engine (EAXREVERB falls back to REVERB if unsupported), create the listener effect slot, the preset map and the two low-pass filters; otherwise force EFX off. Allocate up to 32 hardware sources with the distance model parameters (and an auxiliary send to the listener slot when EFX is on). Doppler factor from config, speed of sound 343.3 m/s.

## Voice allocation — `recomputeSource(sound, reason, value)`

Called whenever a sound's state changes:

```text
audibility = sound.compute_audibility(listener)          (see Sound)
IF audibility == 0: release its hardware voice if any
ELSE IF it has a voice: apply the change (play → update filters then play; stop; gain × master volume; loop; pitch; position; velocity)
ELSE IF a voice is free: assign it
ELSE: find the least audible assigned sound; if quieter than this one, steal its voice
assign = bind buffer, copy gain×master, loop, pitch, position, velocity; play (with filters) if it should be playing
```

**Notes** — the periodic re-ranking of all sounds (`recomputeAllSources`) is compiled out, so voices are only reassigned when some sound changes state; distant looping sounds keep voices until an event touches them.

## `createSound(file, group)`

Reuse the buffer if the file was loaded before; otherwise load it with the built-in WAV reader. Limits reached or load failure → none.

**WAV reader** — "RIFF" … "WAVE", "fmt " chunk ≥ 16 bytes, PCM only (format 1), channels, sample rate, bits per sample; optional "fact" chunk skipped; then "data". Mono 8/16 and stereo 16 are supported; stereo 8-bit is (mis)declared as stereo 16. Anything else fails. (Only positional mono sounds are spatialised by OpenAL; stereo files play non-positionally.)

## `Update(dt)` — once per frame

```text
listener under water? (wavefield)
Doppler: 0 while the simulation is paused (option), else configured
UpdateAlListener: position, velocity, orientation (direction + up)
environment (when the engine controls environmental audio):
  under water → speed of sound 1522 m/s, air absorption 0.00668 (≈0.334 dB/km at 5 kHz in sea water); else 343.3 m/s, 1.0
  EFX: listener reverb preset = GetReverbPresetAt(listener)
EFX per hardware voice: Doppler per source (1, or 0 if its actor is paused and the option is set), air absorption,
  filters (below); listener effect slot update; directed sounds (option)
```

## Reverb presets — `GetReverbPresetAt(p)`

For the listener: a forced preset (config) wins. Otherwise: under water → UNDERWATER; inside a collision box that names a reverb preset (from ODEF files) → that preset; else the configured default preset or none.

The listener's effect slot glides toward the target preset: every parameter moves `min(dt/0.333, 0.5)` of the way per frame (decay HF limit rounded), then the effect is re-applied (EAX or standard reverb parameter set; none → slot emptied).

## Early reflections (EAX engine with reflection panning)

Cast four horizontal rays (90° apart) from the listener, 2 m long, against collision triangles, collision boxes and actor bounding boxes (excluding the vehicle the player sits in). Pan vector = Σ direction × (2 − distance) over hits. With a surface within 2 m: delay = nearest distance / speed of sound; gain = preset gain + 2 − 2·magnitude (≤ max), magnitude = 1 − |pan|/√8; pan rotated into the listener's frame and scaled by magnitude. Otherwise the preset's values.

## Obstruction and occlusion (per assigned voice, EFX)

- *Obstructed* when the direct line to the sound hits a collision triangle, a solid collision box (not containing the sound, beyond 0.1 m of it), another actor's bounding box, or terrain — or always when the listener is inside the bounding box of the vehicle they occupy and "force obstruction inside vehicles" is on. Obstructed direct paths get the low-pass filter (if obstruction is enabled).
- *Occluded* (checked only when obstructed, if occlusion is enabled) when the reverb preset at the sound differs from the listener's: the reverb send gets the wet-path low-pass filter.

## Directed sounds (option)

Sounds attached to exhaust nodes use a cone along exhaust direction (inner 60°, outer 170°, outer gain 0.85, HF 0.80); propeller sounds along the engine axis (170°/270°, 0.85/0.70); turbojet sounds rearward (60°/240°, 0.60/0.60); screw props (70°/170°, 0.80/0.70). Node-0 sounds are left omnidirectional.
