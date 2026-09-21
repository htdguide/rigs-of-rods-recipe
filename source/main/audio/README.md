# source/main/audio

> Vehicle and world sound: data-driven sound scripts, a logical-sound layer over a small pool of hardware voices, an environmental effects model, and positional voice chat.

Three layers. [`SoundScriptManager`](SoundScriptManager.h.md) parses `.soundscript` templates and instantiates them per actor, turning simulation triggers (engine on, horn, brake...) and modulation sources (RPM, speed, airspeed...) into start/stop/pitch/gain changes on [`Sound`](Sound.h.md) objects. Each `Sound` is a logical sound that may or may not currently own one of 32 hardware voices; [`SoundManager`](SoundManager.h.md) hands voices to the most audible sounds and models reverb zones, obstruction, occlusion and underwater propagation. [`MumbleIntegration`](MumbleIntegration.h.md) is separate: it tells an external voice-chat client where the player stands.

All of it sits on [Seam: Positional audio](../../../SYSTEM-REQUIREMENTS.md#seam-positional-audio); the game runs silent when no audio device opens.

## Reading order

1. [`SoundManager`](SoundManager.h.md) ([impl](SoundManager.cpp.md))
2. [`Sound`](Sound.h.md) ([impl](Sound.cpp.md))
3. [`SoundScriptManager`](SoundScriptManager.h.md) ([impl](SoundScriptManager.cpp.md))
4. [`MumbleIntegration`](MumbleIntegration.h.md) ([impl](MumbleIntegration.cpp.md))

## Cycle

`Sound` and `SoundManager` refer to each other: a sound asks the manager to recompute its voice whenever its state changes, and the manager reads the sound's audibility. Read `SoundManager` first as the owner.
