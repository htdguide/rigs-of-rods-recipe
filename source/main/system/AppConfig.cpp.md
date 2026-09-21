# source/main/system/AppConfig.cpp

> Reads and writes `RoR.cfg`, translating enum settings to and from fixed English labels.

**Needs** — [`Console.h`](Console.h.md) · [`Application.h`](../Application.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`resources/ContentManager.h`](../resources/ContentManager.h.md) · [`utils/ErrorUtils.h`](../utils/ErrorUtils.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md) · [`utils/Utils.h`](../utils/Utils.h.md)
**Used by** — callers of [`Console.h`](Console.h.md) — it implements the console's config load/save
**Tier floor** — T4

## Purpose

`RoR.cfg` (in `sys_config_dir`) is user-edited and has existed since before settings were enums, so several values are stored as human-readable labels. This file defines that format exactly.

## State

Stateless; the label tables below are constants.

## File format

Plain lines `key=value` (also `key:value` or `key<TAB>value`), whitespace trimmed, `;` comments. Keys may be short or long setting names. No sections.

## Enum labels

| Setting | Values (in enum order) | Unknown label parses as |
|---|---|---|
| `gfx_shadow_type` | `No shadows (fastest)`, `Parallel-split Shadow Maps` | NONE |
| `gfx_extcam_mode` | `None`, `Static`, `Pitching` | NONE |
| `gfx_texture_filter` | `None (fastest)`, `Bilinear`, `Trilinear`, `Anisotropic (best looking)` | NONE |
| `gfx_vegetation_mode` | `None (fastest)`, `20%`, `50%`, `Full (best looking, slower)` | NONE |
| `sim_gearbox_mode` | `Automatic shift`, `Manual shift - Auto clutch`, `Fully Manual: sequential shift`, `Fully manual: stick shift`, `Fully Manual: stick shift with ranges` | AUTO |
| `gfx_flares_mode` | `None (fastest)`, `No light sources`, `Only current vehicle, main lights`, `All vehicles, main lights`, `All vehicles, all lights` | CURR_VEHICLE_HEAD_ONLY |
| `gfx_water_mode` | `None`, `Basic (fastest)`, `Reflection`, `Reflection + refraction (speed optimized)`, `Reflection + refraction (quality optimized)`, `Hydrax` | BASIC |
| `gfx_sky_mode` | `Sandstorm (fastest)`, `Caelum (best looking, slower)`, `SkyX (best looking, slower)` | SANDSTORM |
| `audio_efx_reverb_engine` | `None (no reverb, fastest)`, `REVERB`, `EAXREVERB (more realistic effects, slower)` | NONE |
| `io_input_grab_mode` (labels defined, **not** used by load/save) | `None`, `All`, `Dynamically` | ALL |

Labels are case-sensitive and must match exactly (note the inconsistent capitalisation in the gearbox labels — it is the format).

## `loadConfig`

```text
FUNCTION load_config()
  IF RoR.cfg missing: RETURN                          # defaults stay
  FOR EACH (key, value) IN file, in order
    key, value = sanitise UTF-8
    cvar = find by short or long name
    IF cvar exists AND NOT archived
      LOG "cannot be set from RoR.cfg"; CONTINUE     # runtime-only settings are protected
    IF cvar missing: cvar = create text setting named key, archived   # keeps unknown keys round-tripping
    parse_value(cvar, value)

FUNCTION parse_value(cvar, value)
  CASE cvar OF
    gfx_envmap_rate:     int clamped to 0..2
    gfx_shadow_quality:  int clamped to 0..3
    enum settings above: label -> enum index (unknown -> fallback in table)
    gfx_fov_external_default, gfx_fov_internal_default: int, ignored if < 10
    otherwise:           typed parse (see CVar.cpp)
```

## `saveConfig`

**Contract** — overwrites `RoR.cfg` in the config resource group. Header lines `; Rigs of Rods configuration file` and `; -------------------------------`, then groups in this order, each introduced by a blank line and `; <Label>`: Application (`app_`), Multiplayer (`mp_`), Simulation (`sim_`), Input/Output (`io_`), Graphics (`gfx_`), GUI (`ui_`), Audio (`audio_`), Diagnostics (`diag_`). Each archived setting whose short name has that prefix is written as `<key>=<value>`, where key is the long name if `app_config_long_names` is true (default) else the short name, and the value is the enum label for the settings above or the setting's text form otherwise. Order within a group follows the registry's hash-map order (unspecified).

**Notes** — archived settings with other prefixes (`flexbody_defrag_*`, `remote_*`, user-created names without a known prefix) are **never written**, so they do not persist even though they are marked archived. A rebuild should either write them or not mark them archived; the original silently drops them.
