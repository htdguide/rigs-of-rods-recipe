# source/main/utils/ConfigFile.cpp

> Typed parsing rules for config values.

**Needs** — [`ConfigFile.h`](ConfigFile.h.md) · [`Application.h`](../Application.h.md) · [`system/Console.h`](../system/Console.h.md) · [`Utils.h`](Utils.h.md)
**Used by** — callers of [`ConfigFile.h`](ConfigFile.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

The value grammar that every INI-style content file relies on.

## State

See header twin.

## Value grammar

**Contract** —
- **string** — the raw value with invalid UTF-8 replaced; an **empty value counts as missing** and yields the default.
- **float / int** — locale-independent decimal parse; unparsable → default.
- **bool** — `true/yes/1` and `false/no/0` (case-insensitive); anything else → default.
- **colour** — `"R G B"` or `"R G B A"`, floats 0–1, whitespace-separated; on failure logs `"<file>: Could not parse '<section>/<key>' (<value>) as color, format must be 'R G B A' or 'R G B', falling back to '<default>'"` and returns the default.
- **vector3** — `"X Y Z"`; same failure behaviour with "as vector3, format must be 'X Y Z'".

## `SetString`

**Contract** — see header twin; used when the editor writes terrain files back.

## `HasSection` / `HasSetting`

**Contract** — lookup in the parsed section map without side effects.
