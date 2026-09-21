# source/main/utils/BitFlags.h

> 32- and 64-bit flag words with 1-based bit numbering.

**Needs** — nothing
**Used by** — [`Application.h`](../Application.h.md) · [`gameplay/ScriptEvents.h`](../gameplay/ScriptEvents.h.md) · [`network/RoRnet.h`](../network/RoRnet.h.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/SimData.h`](../physics/SimData.h.md) · [`physics/flex/FlexFactory.h`](../physics/flex/FlexFactory.h.md) · [`resources/rig_def_fileformat/RigDef_File.h`](../resources/rig_def_fileformat/RigDef_File.h.md) · [`resources/rig_def_fileformat/RigDef_Node.h`](../resources/rig_def_fileformat/RigDef_Node.h.md) · [`system/CVar.h`](../system/CVar.h.md) · [`GenericFileFormat.h`](GenericFileFormat.h.md)
**Tier floor** — T4

## Purpose

Defines the flag-word types used across the program and, importantly, in the network protocol and content formats (`RoRnet` net/light masks, actor state flags, exception-handling flags).

## State

```text
BitMask   = int (32-bit unsigned)
BitMask64 = int (64-bit unsigned)
```

## `BITMASK(n)`

**Contract** — the flag with bit number `n`, **counting from 1**: `BITMASK(1) = 0x1`, `BITMASK(2) = 0x2`, `BITMASK(11) = 0x400`. `BITMASK64` is the same for 64-bit words.

**Notes** — the 1-based numbering is load-bearing: every flag enum in the program, including the RoRnet `Netmask`/`Lightmask` bits sent on the wire, is defined through it. A rebuild using 0-based shifts must subtract one.

## Predicates and setters

**Contract** — `is_all_clear(word, flags)` is true when none of `flags` is set; `is_all_set(word, flags)` when every one is set; `set_clear(word, flags)`, `set_set(word, flags)`, and `set(word, flag, bool)` mutate the word.
