# source/main/system/AppCommandLine.cpp

> Parses program arguments into `cli_*` settings and the help/version exits.

**Needs** — [`Console.h`](Console.h.md) · [`GameContext.h`](../GameContext.h.md) · [`utils/ErrorUtils.h`](../utils/ErrorUtils.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md) · [`utils/Utils.h`](../utils/Utils.h.md)
**Used by** — callers of [`Console.h`](Console.h.md) — it implements `Console::processCommandLine`
**Tier floor** — T4

## Purpose

Command-line options become non-persisted `cli_*` settings which [`main.cpp`](../main.cpp.md) later copies over the matching `diag_preset_*`/`mp_*` values, so the command line wins over `RoR.cfg` for one session only.

## State

Stateless.

## `processCommandLine(argc, argv)`

**Contract** — single-dash options, value separated by a space, except `-joinserver=` which uses `=`. Any parse error or `-help`/`--help` sets `app_state = PRINT_HELP_EXIT`; `-version` sets `PRINT_VERSION_EXIT`; both stop parsing.

| Option | Effect |
|---|---|
| `-map <name>` / `-terrain <name>` | `cli_preset_terrain` |
| `-pos <"x y z">` | `cli_preset_spawn_pos` |
| `-rot <deg>` | `cli_preset_spawn_rot` |
| `-truck <file>` | `cli_preset_vehicle` |
| `-truckconfig <section>` | `cli_preset_veh_config` |
| `-enter` | `cli_preset_veh_enter = true` |
| `-runscript <file>` | appended to `cli_custom_scripts` (comma-separated; repeatable) |
| `-resume` | `cli_resume_autosave = true` |
| `-checkcache` | `cli_force_cache_update = true` |
| `-joinserver=<host>:<port>` | `cli_server_host`, `cli_server_port`; also accepts the URI form `rorserver://host:port/` (strip scheme and trailing slash) |

The last `:` separates host from port.

## `showCommandLineUsage` / `showCommandLineVersion`

**Contract** — info message boxes with the option list (example: `RoR.exe -map simple2 -pos '518 0 518' -rot 45 -truck semi.truck -enter`) and the multi-line version string; the version box also prints the compiler version to stdout.
