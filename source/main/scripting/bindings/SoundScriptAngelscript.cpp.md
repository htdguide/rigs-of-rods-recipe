# source/main/scripting/bindings/SoundScriptAngelscript.cpp

> Script access to sound-script templates, instances and raw sounds.

**Needs** — [`ScriptEngine.h`](../ScriptEngine.h.md) · [`audio/Sound.h`](../../audio/Sound.h.md) · [`audio/SoundScriptManager.h`](../../audio/SoundScriptManager.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## Types

- `SoundScriptTemplateClass` — `getNumSounds`, `getSoundName(i)`, `getSoundPitch(i)`, start/stop sound name and pitch, `getName`, `getFileName`, `getGroupName`, `isBaseTemplate`.
- `SoundClass` — `setPitch`, `setGain`, `setPosition`, `setVelocity`, `setLoop`, `setEnabled`, `play`, `stop`, and getters for enabled, playing, audibility, gain, pitch, loop, hardware index, buffer, position, velocity, source index.
- `SoundScriptInstanceClass` — `runOnce`, `setPitch`, `setGain`, `setPosition`, `setVelocity`, `start`, `stop`, `kill`, `getTemplate`, start/stop/looped sounds and their pitch-gain, `getActorInstanceId`, `getInstanceName`.
- Enums `SoundTriggers` (66 values, SS_TRIG_NONE … SS_TRIG_LINKED_COMMAND) and `ModulationSources` (31 values, SS_MOD_NONE … SS_MOD_MUSIC_VOLUME) — names and values as in [`audio/SoundScriptManager.h`](../../audio/SoundScriptManager.h.md).
