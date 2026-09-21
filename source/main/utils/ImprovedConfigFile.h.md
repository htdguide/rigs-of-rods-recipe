# source/main/utils/ImprovedConfigFile.h

> Read/write INI store with typed setters, used for script-private persistent storage.

**Needs** — [`Application.h`](../Application.h.md) · [`ConfigFile.h`](ConfigFile.h.md)
**Used by** — [`scripting/LocalStorage.h`](../scripting/LocalStorage.h.md)
**Tier floor** — T4

## Purpose

Backs [`LocalStorage`](../scripting/LocalStorage.h.md), which gives each script a small persistent key/value file. Unlike `ConfigFile`, it can save.

## State

```text
RECORD ImprovedConfigFile EXTENDS ConfigFile
  separators : text = "="
```

## `loadImprovedCfg(filename, resource_group)`

**Contract** — parse with `=` as the only separator, trimming whitespace around keys and values.

## `saveImprovedCfg(filename, resource_group)`

**Contract** — overwrite the file: for each section in the map's order, write `[name]` (skipped for the unnamed section) followed by one `key=value` line per entry. Lines are truncated at ~2000 bytes. Always returns true.

## `hasSetting(key, section)` · `setSetting(key, value, section)`

**Contract** — existence test; replace-all-values setter creating sections on demand.

## Typed accessors

**Contract** — get/set pairs for bool, real, int, unsigned, long, unsigned long, angle (radians), vector3, 3×3 and 4×4 matrix, quaternion, colour, and string list, each stored as the rendering engine's canonical text form (space-separated components). Getters of missing keys return that type's zero.
