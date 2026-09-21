# source/main/resources/addonpart_fileformat/AddonPartFileFormat.h

> Turns an `.addonpart` file into an extra vehicle module plus tuning tweaks, and detects conflicts between parts.

**Needs** — [`Application.h`](../../Application.h.md) · [`utils/GenericFileFormat.h`](../../utils/GenericFileFormat.h.md) · [`utils/memory/RefCountingObject.h`](../../utils/memory/RefCountingObject.h.md) · [`rig_def_fileformat/RigDef_File.h`](../rig_def_fileformat/RigDef_File.h.md)
**Used by** — [`gui/panels/GUI_TopMenubar.h`](../../gui/panels/GUI_TopMenubar.h.md) · [`physics/ActorSpawner.cpp`](../../physics/ActorSpawner.cpp.md) · [`resources/CacheSystem.cpp`](../CacheSystem.cpp.md) · [`AddonPartFileFormat.cpp`](AddonPartFileFormat.cpp.md)
**Tier floor** — T4

## Purpose

An add-on part is a small mod that attaches to one specific vehicle (matched by GUID): it can **add** props, flexbodies, flares and managed materials, and **change or remove** existing elements of the vehicle by id. Additions become a fake truck-format module merged at spawn; changes are recorded into the vehicle's [`TuneupDef`](../tuneup_fileformat/TuneupFileFormat.h.md). Two parts that tweak the same element conflict. Implementation: [`AddonPartFileFormat.cpp`](AddonPartFileFormat.cpp.md).

## State

```text
RECORD AddonPartConflict
  part1, part2 : cache entry
  keyword      : text      # e.g. "addonpart_tweak_node"
  element_id   : int

RECORD AddonPartUtility
  document, cursor         : GenericDocument, GenericDocContext
  addonpart_entry          : cache entry
  module                   : truck-format Module under construction
  managedmaterials_options : { double_sided }
  tuneup                   : TuneupDef being filled
  silent                   : bool   # suppress logs (used when computing conflicts with throwaway tuneups)
```

## Operations

**Contract** — `TransformToRigDefModule(entry)` → module or nothing on error; `ResolveUnwantedAndTweakedElements(tuneup, entry)`; static `ResetUnwantedAndTweakedElements(tuneup)`, `RecordAddonpartConflicts(p1, p2, out conflicts)`, `CheckForAddonpartConflict(p1, p2, conflicts)`, `DoubleCheckForAddonpartConflict(actor, entry)`. See `.cpp` twin.
