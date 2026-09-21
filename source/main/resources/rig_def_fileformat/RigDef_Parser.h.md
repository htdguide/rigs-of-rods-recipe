# source/main/resources/rig_def_fileformat/RigDef_Parser.h

> Declares the truck-file parser and its per-file state.

**Needs** — [`system/Console.h`](../../system/Console.h.md) · [`RigDef_Prerequisites.h`](RigDef_Prerequisites.h.md) · [`RigDef_File.h`](RigDef_File.h.md) · [`RigDef_SequentialImporter.h`](RigDef_SequentialImporter.h.md)
**Used by** — [`physics/ActorSpawner.h`](../../physics/ActorSpawner.h.md) · [`resources/CacheSystem.cpp`](../CacheSystem.cpp.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.cpp`](../addonpart_fileformat/AddonPartFileFormat.cpp.md) · [`RigDef_Parser.cpp`](RigDef_Parser.cpp.md) · [`RigDef_SequentialImporter.cpp`](RigDef_SequentialImporter.cpp.md)
**Tier floor** — T4

## Purpose

A push parser: the caller feeds lines, then asks for the finished document. Grammar and behaviour: [`RigDef_Parser.cpp`](RigDef_Parser.cpp.md).

## State

```text
RECORD Parser
  # game defaults (kept to reset to)
  game_inertia, game_node_defaults
  # current user defaults (replaced, never mutated, by directives)
  user_inertia, user_beam_defaults, user_node_defaults, default_minimass
  detacher_group             : int
  managed_material_options   : { double_sided }
  # position
  document, root_module, current_module
  line_number                : int
  line                       : bytes (2000)
  tokens                     : up to 100 (start, length) slices of `line`
  current_block              : Keyword
  log_keyword                : Keyword          # for message prefixes
  any_named_node_defined     : bool             # for legacy reference resolution
  current_submesh, current_camera_rail : optional, built across lines
  pending_comment            : DocComment
  importer                   : SequentialImporter
  filename, resource_group   : text
```

## Public operations

**Contract** — `Prepare()` (reset for a new file), `ProcessOgreStream(stream, group)`, `ProcessRawLine(line)` (for callers that assemble text themselves, e.g. add-on parts), `Finalize()`, `GetFile()` → document, `GetSequentialImporter()`. Static helpers shared with the add-on-part and tune-up readers: `ProcessForsetLine`, `IdentifyKeyword`, `IdentifySpecialProp`.

## Limits

Lines longer than 1999 bytes are truncated; lines with more than 100 tokens lose the extra tokens.
