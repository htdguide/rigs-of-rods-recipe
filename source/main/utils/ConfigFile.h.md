# source/main/utils/ConfigFile.h

> Sectioned `key=value` file reader with typed getters, defaults, and UTF-8 sanitising.

**Needs** — [`system/Console.h`](../system/Console.h.md) · [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine) (base INI parser)
**Used by** — [`resources/otc_fileformat/OTCFileFormat.cpp`](../resources/otc_fileformat/OTCFileFormat.cpp.md) · [`resources/terrn2_fileformat/Terrn2FileFormat.cpp`](../resources/terrn2_fileformat/Terrn2FileFormat.cpp.md) · [`terrain/TerrainGeometryManager.h`](../terrain/TerrainGeometryManager.h.md) · [`ConfigFile.cpp`](ConfigFile.cpp.md) · [`ImprovedConfigFile.h`](ImprovedConfigFile.h.md)
**Tier floor** — T4

## Purpose

Many RoR files are INI-like: `[Section]` headers and `key=value` lines (`.terrn2`, `ground_models.cfg`, `.skin` legacy, `RoR.cfg`, `.otc`). This type adds typed lookups with defaults on top of the rendering engine's INI parser, and forbids the base "get raw setting" call because it does not sanitise UTF-8. Implementation: [`ConfigFile.cpp`](ConfigFile.cpp.md).

## State

```text
RECORD ConfigFile
  sections : map<text, multimap<text, text>>   # "" is the section before any header
  log_filename : text                          # prefix for parse warnings
  log_area     : console message area
```

## Typed getters

`getString`, `getFloat`, `getInt`, `getBool`, `getColourValue`, `getVector3` — each takes `(key, section = "", default)`. Contracts in the `.cpp` twin.

## `SetString(key, value, section = "")`

**Contract** — replace all values of `key` in `section` with a single value, creating the section if needed.

## `HasSection(name)` · `HasSetting(section, key)`

**Contract** — existence tests that do not log errors.

## `setLoggingInfo(filename, area)`

**Contract** — sets the prefix used in warnings so the user sees which file had the bad value.
