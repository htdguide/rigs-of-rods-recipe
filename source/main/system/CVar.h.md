# source/main/system/CVar.h

> A named, typed, optionally persisted setting ("console variable") holding a number and its text form.

**Needs** — [`utils/BitFlags.h`](../utils/BitFlags.h.md)
**Used by** — [`Application.h`](../Application.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`CVar.cpp`](CVar.cpp.md) · [`Console.h`](Console.h.md) · [`utils/Utils.cpp`](../utils/Utils.cpp.md)
**Tier floor** — T4

## Purpose

RoR's configuration and much of its cross-module state live in Quake-style console variables: each has a short name (`gfx_water_mode`), an optional long name used in older config files (`Water effects`), a type, and flags. The console, the config file, the command line, scripts and the settings UI all read and write them by name. Registry and defaults: [`CVar.cpp`](CVar.cpp.md).

## State

```text
FLAGS CVarFlags
  TYPE_BOOL  = bit 1
  TYPE_INT   = bit 2
  TYPE_FLOAT = bit 3       # none of the three -> text variable
  ARCHIVE    = bit 4       # persisted to RoR.cfg
  NO_LOG     = bit 5       # value changes are not logged (passwords, tokens)

RECORD CVar
  name      : text
  long_name : text         # equals name when none given
  flags     : int
  value_num : real (32-bit)   # authoritative for numeric types; 0 for text types
  value_str : text            # authoritative for text types; derived for numeric types
```

Invariant: for numeric types, `value_str == format(value_num)`; for text types, `value_num == 0`.

## `setVal(number)`

**Contract** — if the value differs, logs `[RoR|CVar] <name>: "<new>" (was: "<old>")` (unless NO_LOG), then stores the number and its text form. Formatting by type: bool → `Yes`/`No`; int → decimal; float → six-decimal fixed form.

**Notes** — the change test compares after converting the stored float to the argument's type, so setting a float CVar to an int value that truncates equal is a no-op.

## `setStr(text)`

**Contract** — if the text differs, logs, stores the text and zeroes the number. Callers use this only on text variables; numeric variables must be assigned through the console's parser (`cVarAssign`), which parses the text first.

## Getters

**Contract** — `getStr`, `getFloat`, `getInt` (truncating), `getBool` (non-zero), `getEnum<T>` (the int reinterpreted), `getName`, `getLongName`, `hasFlag(mask)` (true if **any** bit of mask is set).
