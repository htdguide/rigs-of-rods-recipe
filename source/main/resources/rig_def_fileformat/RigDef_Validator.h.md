# source/main/resources/rig_def_fileformat/RigDef_Validator.h

> Formal checks on a parsed truck document for one chosen configuration of modules.

**Needs** — [`RigDef_File.h`](RigDef_File.h.md)
**Used by** — [`physics/ActorManager.cpp`](../../physics/ActorManager.cpp.md) · [`RigDef_Validator.cpp`](RigDef_Validator.cpp.md)
**Tier floor** — T4

## Purpose

Runs between parsing and spawning. It validates the *combination* of modules the player selected (the root module plus chosen `section`s) and removes individually invalid lines so the spawner never sees them. Implementation: [`RigDef_Validator.cpp`](RigDef_Validator.cpp.md).

## State

```text
RECORD Validator
  document         : Document
  selected_modules : list<Module>     # root first, then added modules
  check_beams      : bool = true      # currently unused by any check
```

## `Setup(document)` · `AddModule(name)` · `Validate()` · `SetCheckBeams(bool)`

**Contract** — setup selects the root module; add-module selects a user module by name (false if absent); validate returns overall validity and edits the selected modules in place.
