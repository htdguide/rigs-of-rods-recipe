# source/main/system/ConsoleCmd.h

> Base type of a console command: name, usage text, description, and a run action.

**Needs** — [`Application.h`](../Application.h.md) · [`utils/Language.h`](../utils/Language.h.md)
**Used by** — [`Console.h`](Console.h.md) · [`ConsoleCmd.cpp`](ConsoleCmd.cpp.md)
**Tier floor** — T4

## Purpose

Commands are objects in a name-keyed map so scripts and future modules can add more. Built-ins: [`ConsoleCmd.cpp`](ConsoleCmd.cpp.md).

## State

```text
RECORD ConsoleCmd
  name, usage, doc : text
  run : (args: list<text>) -> nothing     # args[0] is the command name
```

## `CheckAppState(state)`

**Contract** — true if the program is in `state`; otherwise prints an error reply ("Only allowed when simulation is running" for SIMULATION, "Not allowed in current app state" otherwise) and returns false.
