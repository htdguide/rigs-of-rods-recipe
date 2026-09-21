# source/main/Application.cpp

> Storage for the global settings and singleton subsystems, the factories that create them, exception triage, logging, and enum→text tables.

**Needs** — [`Application.h`](Application.h.md) · [`AppContext.h`](AppContext.h.md) · [`resources/CacheSystem.h`](resources/CacheSystem.h.md) · [`gfx/camera/CameraManager.h`](gfx/camera/CameraManager.h.md) · [`system/Console.h`](system/Console.h.md) · [`resources/ContentManager.h`](resources/ContentManager.h.md) · [`network/DiscordRpc.h`](network/DiscordRpc.h.md) · [`GameContext.h`](GameContext.h.md) · [`gfx/GfxScene.h`](gfx/GfxScene.h.md) · [`gui/GUIManager.h`](gui/GUIManager.h.md) · [`utils/InputEngine.h`](utils/InputEngine.h.md) · [`utils/Language.h`](utils/Language.h.md) · [`network/OutGauge.h`](network/OutGauge.h.md) · [`gui/OverlayWrapper.h`](gui/OverlayWrapper.h.md) · [`audio/MumbleIntegration.h`](audio/MumbleIntegration.h.md) · [`network/Network.h`](network/Network.h.md) · [`scripting/ScriptEngine.h`](scripting/ScriptEngine.h.md) · [`audio/SoundScriptManager.h`](audio/SoundScriptManager.h.md) · [`terrain/Terrain.h`](terrain/Terrain.h.md) · [`threadpool/ThreadPool.h`](threadpool/ThreadPool.h.md)
**Used by** — callers of [`Application.h`](Application.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

The definitions behind [`Application.h`](Application.h.md). Separated only because the source language splits declaration from definition.

## State

```text
# Always present from program start (constructed before main runs):
app_context, cache_system, console, content_manager, discord_rpc,
game_context, gfx_scene (uninitialised), language_engine, out_gauge,
network (only if multiplayer is compiled in)

# Created on demand by App::Create…() and otherwise absent:
camera_manager, gui_manager, input_engine, mumble, overlay_wrapper,
script_engine, sound_script_manager, thread_pool

# ~200 CVar handles, filled in by the console's CVar registration (system/CVar.cpp)
```

Invariant: each `Create…` may run at most once (asserted); only `overlay_wrapper` and `input_engine` have matching `Destroy…`.

## `App::Create…`

**Contract** — construct the named singleton. Each is conditional on its feature being compiled in: Mumble on positional-voice support, sound-script manager on audio, script engine on scripting. `CreateGfxScene` does not construct but initialises the always-present scene (creates the scene manager). `CreateThreadPool` delegates to [`ThreadPool::DetectNumWorkersAndCreate`](threadpool/ThreadPool.h.md#detectnumworkersandcreate).

## `HandleGenericException`

```text
FUNCTION handle_generic_exception(from: text, flags)
  error = the error currently being handled
  message = CASE error OF
              engine error   -> its description
              standard error -> its text
              anything else  -> "Unknown exception"
  IF CONSOLE IN flags:  console.put(INFO, SYSTEM_ERROR, from + ": " + message)   # console also logs
  ELSE IF LOGFILE IN flags: log(from + ": " + message)
  IF SCRIPTEVENT IN flags AND scripting is compiled in:
    script_engine.forward_exception_as_script_event(from)
```

**Notes** — console output already reaches the log file, so CONSOLE suppresses the explicit LOGFILE write to avoid duplicate lines.

## `Log` / `LogFormat`

**Contract** — `Log` hands the line to the rendering engine's log manager, whose default log is `RoR.log` (set up in [`AppContext::SetUpLogging`](AppContext.cpp.md#setuplogging)); the console is a listener on that log, so every logged line also reaches the in-game console filter. `LogFormat` formats into a 2000-byte buffer first.

## `ToLocalizedString` (one per settings enum)

**Contract** — returns the translated display label for each value, using the translation context named after the enum (e.g. context `"GfxWaterMode"`, text `"Reflection + refraction (speed optimized)"`). Unknown values return empty text. Labels are listed with the enums in [`Application.h`](Application.h.md#settings-enums).

## `MsgTypeToString`, `TObjSpecialObjectToString`, `RigDef::KeywordToString`

**Contract** — value → canonical spelling tables. Message names are the enum identifiers verbatim; tobj names and truck keywords are the lowercase spellings used in files. `KeywordToString` does not cover every keyword (some newer ones return empty); callers only use it for diagnostics and the serializer, which spells keywords itself.
