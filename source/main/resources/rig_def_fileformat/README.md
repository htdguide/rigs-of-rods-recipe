# source/main/resources/rig_def_fileformat — the truck file format

Part of chapter 7. The single most important data format in RoR: every vehicle, load, boat and aircraft mod is a truck-family file (`.truck .car .boat .airplane .train .trailer .load .fixed`). Depends on the hub's `Keyword` enum and on physics constants for defaults; consumed by the [actor spawner](../../physics/ActorSpawner.cpp.md), the [mod cache](../CacheSystem.cpp.md) (metadata only), the [add-on part](../addonpart_fileformat/README.md) and [tune-up](../tuneup_fileformat/README.md) systems, and the [actor exporter](../../physics/ActorExport.cpp.md).

Pipeline: **Parser** (text → [document](RigDef_File.h.md)) → **SequentialImporter** (legacy node indices → modern layout) → **Validator** (per chosen configuration) → spawner. **Serializer** goes back to text.

The format in one paragraph: line-based text; first data line is the vehicle's title; `;` or `/` at line start is a comment; separators are space, tab, `,`, `:`, `|`; a keyword alone on a line starts a **block** whose following lines are records; keywords followed by arguments are **directives** applied immediately; `set_*_defaults` directives affect only subsequent lines; `section <n> <name>` … `end_section` groups lines into optional **modules** (configurations); nodes are numbered (`nodes`) or named (`nodes2`) and everything else refers to them by id.

| Twin | Role |
|---|---|
| [`RigDef_Prerequisites.h`](RigDef_Prerequisites.h.md) | Forward declarations |
| [`RigDef_Node.h`](RigDef_Node.h.md) · [`.cpp`](RigDef_Node.cpp.md) | Node ids, references, ranges, node options |
| [`RigDef_File.h`](RigDef_File.h.md) · [`.cpp`](RigDef_File.cpp.md) | Document/module model, every record and its defaults |
| [`RigDef_Regexes.h`](RigDef_Regexes.h.md) | Keyword recogniser, axles pattern |
| [`RigDef_Parser.h`](RigDef_Parser.h.md) · [`.cpp`](RigDef_Parser.cpp.md) | **Grammar: per-section column syntax, directives, quirks** |
| [`RigDef_SequentialImporter.h`](RigDef_SequentialImporter.h.md) · [`.cpp`](RigDef_SequentialImporter.cpp.md) | Legacy node renumbering |
| [`RigDef_Validator.h`](RigDef_Validator.h.md) · [`.cpp`](RigDef_Validator.cpp.md) | Acceptance rules |
| [`RigDef_Serializer.h`](RigDef_Serializer.h.md) · [`.cpp`](RigDef_Serializer.cpp.md) | Text output (with known defects) |

The original `ReadMe.txt` (a technical spec from RoR 0.4.5) is folded into the parser twin; its most useful remaining facts: `sectionconfig` is ignored, `end` is optional, order of sections does not matter except that `add_animation`, `prop_camera_mode` must follow a prop and `forset`/`forvert`/`flexbody_camera_mode` must follow a flexbody. Official user documentation: https://docs.rigsofrods.org/vehicle-creation/fileformat-truck.
