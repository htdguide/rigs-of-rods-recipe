# source/main/system/ConsoleCmd.cpp

> The built-in console commands and the line dispatcher.

**Needs** — [`Console.h`](Console.h.md) · [`ConsoleCmd.h`](ConsoleCmd.h.md) · [`Application.h`](../Application.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`gameplay/Character.h`](../gameplay/Character.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gfx/IGfxWater.h`](../gfx/IGfxWater.h.md) · [`network/Network.h`](../network/Network.h.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`terrain/TerrainObjectManager.h`](../terrain/TerrainObjectManager.h.md) · [`audio/SoundManager.h`](../audio/SoundManager.h.md)
**Used by** — callers of [`ConsoleCmd.h`](ConsoleCmd.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

What the player can type into the console, and how a typed line is routed.

## State

Stateless beyond the command map.

## `doCommand(line)`

```text
FUNCTION do_command(line)
  IF line starts with '/' or '\'
    print HELP "Using slashes before commands are deprecated…"; drop the first char
  IF line starts with '!'
    send line as multiplayer chat (server-side commands like "!kick"); RETURN
  args = split line on single spaces
  IF args[0] names a command: run it with args; RETURN
  IF args[0] names a setting (short or long name): reply "<name> = <value>"; RETURN
  reply ERROR "unknown command: <line>"
```

## Built-in commands

Replies go to the console as SYSTEM_REPLY (or SYSTEM_ERROR / HELP). "sim" = only in SIMULATION state.

| Command | Usage | Behaviour |
|---|---|---|
| `gravity` | `[<number>\|earth\|moon\|mars\|jupiter]` | sim; get or set terrain gravity (m/s², negative = down): earth −9.807, moon −1.62, mars −3.711, jupiter −24.8 |
| `waterlevel` | `[<number>\|default]` | sim; error if the terrain has no water; sets static water height (`default` = the terrain's configured height); replies with the current height |
| `terrainheight` | — | sim; terrain height under the player's vehicle, or the character if on foot |
| `spawnobject` | `<odef name>` | sim; places a terrain object (`.odef`) at the character's position, zero rotation, instance name "Console" |
| `log` | — | toggles `diag_log_console_echo` |
| `ver` | — | "Rigs of Rods <version> (RoRnet_2.45) [date, time]" |
| `pos` | — | sim; position of vehicle or character |
| `goto` | `<x> <y> <z>` | sim; moves the character, or resets the vehicle to that position (keeping orientation) and fires the script event "truck teleport" |
| `as` | `<code…>` | sim; first fires script event "AngelScript manipulation: console snippet executed" (anti-cheat hook), echoes `>>> code`, then executes the joined arguments as script code |
| `speedofsound` | — | sim; the audio engine's current speed of sound |
| `quit` | — | pushes the shutdown message |
| `help` | — | lists every command as `name usage - doc`, then a tip about arrow-key history |
| `clear` | `[all\|info\|net\|chat\|terrn\|actor\|script]` | removes messages: all; by area; `chat` = NETCHAT type; `net` = any with a non-zero user id |
| `loadscript` | `<filename>` | loads a script file as a "custom" script unit; replies with its id or a failure hint |
| `set` | `<cvar> [<value>]` | get/set an **existing** setting by short or long name |
| `setstring` / `setbool` / `setint` / `setfloat` | `[--archive] <cvar> [<value>]` | get or **create** a setting of that type; `--archive` marks it persistent |
| `vars` | `[<substr>…]` | lists settings whose short name contains any substring (all if none): `name=value (type[, archive][, no log])` |
