# source/main/resources/terrn2_fileformat/Terrn2FileFormat.cpp

> Reads `.terrn2` (INI) files.

**Needs** — [`Terrn2FileFormat.h`](Terrn2FileFormat.h.md) · [`AppContext.h`](../../AppContext.h.md) · [`utils/ConfigFile.h`](../../utils/ConfigFile.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md) · [`physics/SimConstants.h`](../../physics/SimConstants.h.md)
**Used by** — callers of [`Terrn2FileFormat.h`](Terrn2FileFormat.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Defines the `.terrn2` keys and their defaults.

## State

Stateless.

## Format

INI via [`ConfigFile`](../../utils/ConfigFile.cpp.md) with separators tab, `:`, `=`; whitespace trimmed. Errors are reported to the console with area TERRN.

| Section / key | Field | Default |
|---|---|---|
| `[General] Name` | name | **required** (empty → error "Terrain name is empty", no document) |
| `GeometryConfig` | .otc file | empty; if present must contain `.otc` else error, no document |
| `AmbientColor` | "r g b [a]" | white |
| `CategoryID` | int | 129 |
| `GUID` | text | "" |
| `Version` | int | 1 |
| `Gravity` | real | −9.81 (note: not the −9.807 physics default) |
| `CaelumConfigFile` | text | "" |
| `SandStormCubeMap` | text (skybox material) | "" |
| `CaelumFogStart`, `CaelumFogEnd` | int | −1 |
| `Water` | bool | false |
| `HydraxConfigFile`, `SkyXConfigFile` | text | "" |
| `TractionMap` | text | "" |
| `WaterLine`, `WaterBottomLine` | real | 0 |
| `CustomMaterial` | text | "" |
| `StartPosition` | "x y z" | 512 0 512 |
| `StartRotation` | degrees | 0; presence recorded |
| `[Authors] <type>=<name>` | authors | entries with empty name skipped |
| `[Objects] <file>=` | `.tobj` files | keys are the file names |
| `[Scripts] <file>=` | `.as` files | |
| `[AssetPacks] <file>=` | asset packs | |
| `[AI Presets] <file>=` | waypoint preset JSON files | |
| `[Teleport] NavigationMapImage` | image | "" |
| `[Teleport] Telepoint<N>/Position`, `Telepoint<N>/Name` | telepoints | N = 1, 2, 3 … until a Position is missing |

Telepoint positions must parse as `x, y, z` (comma-separated); a malformed one is skipped with a warning but the numbering continues.
