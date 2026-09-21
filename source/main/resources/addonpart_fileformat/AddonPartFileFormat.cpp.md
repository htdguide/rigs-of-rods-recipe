# source/main/resources/addonpart_fileformat/AddonPartFileFormat.cpp

> The `.addonpart` grammar, how additions become a module, how tweaks are recorded, and conflict detection.

**Needs** — [`AddonPartFileFormat.h`](AddonPartFileFormat.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`Application.h`](../../Application.h.md) · [`resources/CacheSystem.h`](../CacheSystem.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`utils/GenericFileFormat.h`](../../utils/GenericFileFormat.h.md) · [`gui/panels/GUI_MessageBox.h`](../../gui/panels/GUI_MessageBox.h.md) · [`rig_def_fileformat/RigDef_Parser.h`](../rig_def_fileformat/RigDef_Parser.h.md) · [`tuneup_fileformat/TuneupFileFormat.h`](../tuneup_fileformat/TuneupFileFormat.h.md)
**Used by** — callers of [`AddonPartFileFormat.h`](AddonPartFileFormat.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Defines the add-on part format and its merge semantics.

## State

See header twin.

## File format

Read with the [generic tokeniser](../../utils/GenericFileFormat.cpp.md) using `//` comments and naked strings. Header lines (`addonpart_name`, `addonpart_description`, `addonpart_guid`, `addonpart_filename`, `addonpart_preview`, `addonpart_author`, `addonpart_category`… — read by the [mod cache](../CacheSystem.cpp.md)) are ignored here. All node references are **numbers** in the target vehicle's final node order.

**Additions** — the truck-format block keywords `managedmaterials`, `props`, `flexbodies`, `flares`, `flares2` open blocks, and `set_managedmaterials_options <0|1>` is honoured; other truck keywords are ignored.

| Block | Line |
|---|---|
| `managedmaterials` | `name type diffuse [specular] [damaged_diffuse]` — note the order differs from truck files (specular before damaged); `-` means none |
| `props` | `ref x y ox oy oz rx ry rz mesh [special args as in truck files]` (≥ 10) |
| `flexbodies` | same 10 columns, and the **next line** must be a `forset` string; ranges are expanded inclusive directly (no legacy importer) |
| `flares` | `ref x y ox oy [type] [control/dashlink] [blink] [size] [material]` (≥ 5; offset z stays 1) |
| `flares2` | `ref x y ox oy oz [type] …` (≥ 6) |

Lines with too few arguments are skipped with a warning.

**Directives** (line starts with the keyword):

| Directive | Arguments | Effect (unless the element is protected in the tuneup) |
|---|---|---|
| `addonpart_unwanted_prop` | id | mark prop unwanted |
| `addonpart_unwanted_flexbody` | id | mark flexbody unwanted |
| `addonpart_unwanted_flare` | id | mark flare unwanted |
| `addonpart_unwanted_exhaust` | id | mark exhaust unwanted |
| `addonpart_unwanted_managedmaterial` | name | mark material unwanted |
| `addonpart_tweak_node` | id x y z | override node position |
| `addonpart_tweak_cinecam` | id x y z | override cinecam position |
| `addonpart_tweak_wheel` | id media1 [media2 [side(l/r) [tire_radius [rim_radius]]]] | optional args stop at the first missing/ill-typed one |
| `addonpart_tweak_prop` | id ox oy oz rx ry rz media1 [media2] | all but media2 required |
| `addonpart_tweak_flexbody` | id ox oy oz rx ry rz media | all required |
| `addonpart_tweak_managedmaterial` | name type [media1 [media2 [media3]]] | |

Ill-typed arguments produce "bad arguments" and the directive is skipped.

## `TransformToRigDefModule(entry)`

**Contract** — loads the part's resources, tokenises the file, and builds a module named after the part's display name, with `origin_addonpart` set so the spawner loads meshes from the part's resource group. Returns nothing (with a warning) if the file cannot be read.

## `ResolveUnwantedAndTweakedElements(tuneup, entry)`

**Contract** — applies the directives above to `tuneup`. The first part to tweak an element wins; if a *different* part later tweaks the same element, the tweak is **removed entirely** (with a warning "Resetting tweaks … due to conflict") rather than letting either win — conflicting parts cancel each other out. The same part tweaking twice keeps its first tweak.

```text
FUNCTION record_tweak(map, id, part, data)
  IF id is protected: log skip; RETURN
  IF id not in map: map[id] = data (origin = part)
  ELSE IF map[id].origin != part: log conflict; REMOVE map[id]
```

## `ResetUnwantedAndTweakedElements(tuneup)`

**Contract** — clears unwanted props/flexbodies/flares and node/cinecam/wheel/prop/flexbody tweaks, before re-resolving all installed parts (e.g. after the player changes the part list). Unwanted exhausts/materials and material tweaks are **not** cleared — an original omission.

## Conflict detection

```text
FUNCTION record_conflicts(p1, p2, out conflicts)
  FOR EACH part IN (p1, p2)
    IF part.addonpart_data_only is empty
      part.addonpart_data_only = new tuneup; resolve(part.addonpart_data_only, part) silently
  FOR EACH kind IN (node, wheel, prop, flexbody)
    FOR EACH id tweaked by p1 of this kind
      IF p2 also tweaks id: conflicts.append({p1, p2, "addonpart_tweak_<kind>", id})
```

Cinecam and managed-material tweaks and "unwanted" directives are not considered conflicts. `CheckForAddonpartConflict(p1, p2, conflicts)` answers whether a pair already appears (in either order). `DoubleCheckForAddonpartConflict(actor, part)` checks a requested part against every part installed on the actor and, if any conflict exists, pushes a message box ("Cannot install addon part, conflicts were detected.", listing each conflict as `[i/N] '<part>' (file '<file>') conflicts with '<keyword>' #<id>.`) and returns true.
