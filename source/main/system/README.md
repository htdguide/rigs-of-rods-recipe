# source/main/system — settings, console, config file, command line

Chapter 6. Depends on the hub and on `utils/`; everything later reads settings through it.

The central idea is the **CVar**: a named, typed global setting. There are ~185 built-in ones. `RoR.cfg`, the command line, the console, the settings UI and scripts are five different doors onto the same registry. The console object also keeps the message history that the console window, the chat box and on-screen notifications display.

Startup order (driven by [`main.cpp`](../main.cpp.md)): create built-in CVars → parse command line → establish directories → load `RoR.cfg` → apply `cli_*` overrides.

| Twin | Role |
|---|---|
| [`CVar.h`](CVar.h.md) | Setting record, flags, text/number duality |
| [`CVar.cpp`](CVar.cpp.md) | Registry operations + **table of all built-in settings with defaults** |
| [`Console.h`](Console.h.md) | Console object: message history, maps, entry points |
| [`Console.cpp`](Console.cpp.md) | Message logging and log echo |
| [`ConsoleCmd.h`](ConsoleCmd.h.md) | Command base type |
| [`ConsoleCmd.cpp`](ConsoleCmd.cpp.md) | Built-in commands, line dispatch |
| [`AppCommandLine.cpp`](AppCommandLine.cpp.md) | Program arguments |
| [`AppConfig.cpp`](AppConfig.cpp.md) | `RoR.cfg` format and enum labels |
