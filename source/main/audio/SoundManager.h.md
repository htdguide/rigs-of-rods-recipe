# source/main/audio/SoundManager.h

> The OpenAL layer: device/context, 32 hardware voices shared by up to 8192 logical sounds, WAV loading, and environmental effects.

**Needs** — [`physics/Actor.h`](../physics/Actor.h.md) · [`Application.h`](../Application.h.md) · [`physics/collision/Collisions.h`](../physics/collision/Collisions.h.md) · [`GameContext.h`](../GameContext.h.md) · [`Sound.h`](Sound.h.md) · [`SoundScriptManager.h`](SoundScriptManager.h.md)
**Used by** — [`Sound.cpp`](Sound.cpp.md) · [`SoundManager.cpp`](SoundManager.cpp.md) · [`SoundScriptManager.cpp`](SoundScriptManager.cpp.md) · [`SoundScriptManager.h`](SoundScriptManager.h.md) · [`gui/panels/GUI_GameSettings.cpp`](../gui/panels/GUI_GameSettings.cpp.md) · [`system/ConsoleCmd.cpp`](../system/ConsoleCmd.cpp.md)
**Tier floor** — T2


## Purpose

Wraps the [Seam: Positional audio](../../../SYSTEM-REQUIREMENTS.md#seam-positional-audio). Implementation: [`SoundManager.cpp`](SoundManager.cpp.md). Compiled only with audio support.

## State

```text
CONSTANTS: max distance 500 m, rolloff 1, reference distance 7.5 m, 32 hardware sources, 8192 buffers/sounds
RECORD SoundManager
  device, context; hardware sources[≤32] + map hardware → sound (−1 free) + in-use count
  sounds[≤8192] (logical), buffers[≤8192] + file names (a buffer per distinct file, shared)
  listener: position, direction, up, velocity, under water
  EFX: available, listener effect slot, obstruction low-pass filter (gain 0.33, HF 0.25), occlusion wet-path filter (same),
       air absorption factor, reverb engine (NONE | REVERB | EAXREVERB), current listener reverb preset,
       preset map (name → properties; the standard EFX preset list), effect per slot
```

## API

`createSound(file, group?) → sound?`, `Update(dt)`, `SetListener`, `pauseAllSounds` / `resumeAllSounds` (listener gain 0 / master volume), `setMasterVolume`, `isDisabled` (no device), speed of sound and Doppler get/set, air absorption, `GetEfxProperties(name)`, `GetReverbPresetAt(pos)`, `CleanUp`.
