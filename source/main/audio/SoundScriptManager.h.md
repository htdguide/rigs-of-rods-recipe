# source/main/audio/SoundScriptManager.h

> Sound scripts: the .soundscript format, template/instance model, and the trigger/modulation routing the whole game uses to make sound.

**Needs** — [`scripting/bindings/AngelScriptBindings.h`](../scripting/bindings/AngelScriptBindings.h.md) · [`Application.h`](../Application.h.md) · [`utils/memory/RefCountingObjectPtr.h`](../utils/memory/RefCountingObjectPtr.h.md) · [`Sound.h`](Sound.h.md) · [`SoundManager.h`](SoundManager.h.md)
**Used by** — [`Application.cpp`](../Application.cpp.md) · [`GameContext.cpp`](../GameContext.cpp.md) · [`SoundManager.h`](SoundManager.h.md) · [`SoundScriptManager.cpp`](SoundScriptManager.cpp.md) · [`gameplay/AutoPilot.cpp`](../gameplay/AutoPilot.cpp.md) · [`gameplay/Engine.cpp`](../gameplay/Engine.cpp.md) · [`gameplay/TyrePressure.cpp`](../gameplay/TyrePressure.cpp.md) · [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`gui/OverlayWrapper.cpp`](../gui/OverlayWrapper.cpp.md) · [`main.cpp`](../main.cpp.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorForcesEuler.cpp`](../physics/ActorForcesEuler.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/air/TurboJet.cpp`](../physics/air/TurboJet.cpp.md) · [`physics/air/TurboProp.cpp`](../physics/air/TurboProp.cpp.md) · [`physics/water/ScrewProp.cpp`](../physics/water/ScrewProp.cpp.md) · [`resources/ContentManager.cpp`](../resources/ContentManager.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`scripting/bindings/SoundScriptAngelscript.cpp`](../scripting/bindings/SoundScriptAngelscript.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md)
**Tier floor** — T2


## Purpose

Game code never plays files. It fires **triggers** (start/stop/once/kill/toggle) and sets **modulation** values (rpm, speed…) for an actor; sound-script *instances* bound to that actor and trigger turn this into layered, pitch-crossfaded sounds. Content defines the mapping in `.soundscript` files. Implementation: [`SoundScriptManager.cpp`](SoundScriptManager.cpp.md); low-level playback in [`SoundManager`](SoundManager.h.md). Without audio support compiled in, all trigger/modulate calls compile to nothing.

## State

```text
LIMITS: 16 sounds per script; 256 instances per trigger or modulation source
TRIGGERS (id order): engine, aeroengine1..4, horn, brake, pump, starter, turbo_BOV, turbo_waste_gate, turbo_back_fire, always_on,
  repair, air, gpws_ap_disconnect, gpws_10/20/30/40/50/100, gpws_pull_up, gpws_minimums, air_purge, shift, gear_slide, creak,
  break, screetch, parking_brake, afterburner1..8, aeroengine5..8, aoa_horn, ignition, reverse_gear, turn_signal,
  turn_signal_tick, turn_signal_warn_tick, antilock, tractioncontrol, avionic_chat_01..13, linked_command, main_menu
MODULATIONS: none, engine_rpm, turbo_rpm, aeroengine1..4_rpm, wheel_speed_kmph, injector_ratio, torque_nm, gearbox_rpm, creak,
  break, screetch, pump_rpm, aeroengine1..8_throttle, aeroengine5..8_rpm, air_speed_knots, angle_of_attack_degree,
  linked_command_rate, music_volume
LINK TYPES: default, command, hydro, collision, shocks, brakes, ropes, ties, particles, axles, flares, flexbodies, exhausts, videocamera

RECORD SoundScriptTemplate
  name, file, group, base (loaded from the base resource set), trigger source, gain source + (offset, multiplier, square),
  pitch source + (offset, multiplier, square), sounds[≤16] with reference pitches (ascending; 0 = unpitched),
  optional start sound + pitch, optional stop sound + pitch, unpitchable flag

RECORD SoundScriptInstance
  actor id (or −1 unknown, −2 terrain object), template, name "<file>-<actor>-<counter>", link type + item id,
  start/stop/loop sounds, per-sound crossfade gains, last gain

RECORD SoundScriptManager
  disabled, loading_base, distances (max 500, reference 7.5, rolloff 1), instance counter
  templates : map<name, template>; instances : list
  routing tables: per trigger → instances; per modulation → gain instances, pitch instances
  trigger states : map<link type, item, actor, trigger> → on/off
```

## API

Script loading (pattern `*.soundscript`, loading order 1000), `createInstance(template, actor, link type, item) → instance?`, `removeInstance`, trigger ops for an actor id or actor handle, `getTrigState`, `modulate(actor, source, value)`, `setEnabled` (pause/resume all), listener, `update(dt)`, base-sound flag.
