# source/main/resources/tuneup_fileformat/TuneupFileFormat.cpp

> Tweak resolution helpers and the `.tuneup` text format.

**Needs** — [`TuneupFileFormat.h`](TuneupFileFormat.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`Application.h`](../../Application.h.md) · [`resources/CacheSystem.h`](../CacheSystem.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md)
**Used by** — callers of [`TuneupFileFormat.h`](TuneupFileFormat.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

The resolution rules for tuned values, and the parser/writer of `.tuneup` files.

## State

Stateless.

## Resolution rules

```text
tire/rim radius  = tweak.value IF tweak exists AND tweak.value > 0 ELSE original
wheel/prop/flexbody/managed-material media[i]
                 = tweak.media[i] IF tweak exists AND non-empty ELSE original
node / cinecam position, prop / flexbody offset & rotation
                 = tweak value IF tweak exists ELSE original
managed material type = tweak.type IF non-empty ELSE original
wheel side       = forced side IF forced
                   ELSE tweak.side IF tweak exists AND side != INVALID
                   ELSE original
video camera role= forced role IF forced ELSE original      # add-on tweaks of roles not implemented
anyhow removed   = unwanted OR force-removed
```

**Resource group of tweaked media** (`…MediaRG`): when a tweak supplies media, the files live inside the add-on part's archive, so the resource group to load them from is the add-on part's cache entry group (looked up by the tweak's origin filename). If the add-on part is not in the cache, a warning is logged and the caller's original group is used.

Protection is **not** checked here; the add-on part importer refuses to record tweaks for protected elements (see [`AddonPartFileFormat.cpp`](../addonpart_fileformat/AddonPartFileFormat.cpp.md)).

## `clone`

**Contract** — copies info fields, used add-on parts, prop/flexbody/wheel/node sets and tweaks, forced video-camera roles, and the flare/exhaust/managed-material protection sets. It does **not** copy cinecam tweaks, managed-material tweaks, unwanted flares/exhausts/managed materials, or forced removals of flares/exhausts/managed materials — those are recomputed from add-on parts at spawn, or are session-only UI state. A rebuild that persists them must extend the file format too.

## `.tuneup` format

Same block grammar as `.skin`:

```text
<tuneup name>
{
    preview = <image>
    description = <text>
    author_name = <text>
    author_id = <int>
    category_id = <int>
    guid = <target vehicle guid>          # stored trimmed, lowercased
    filename = <target vehicle file>
    name = <text>
    use_addonpart = <file.addonpart>      # repeatable
    unwanted_prop = <prop id>             # repeatable
    unwanted_flexbody = <flexbody id>
    protected_prop = <prop id>
    protected_flexbody = <flexbody id>
    force_remove_prop = <prop id>
    force_remove_flexbody = <flexbody id>
    forced_wheel_side = <wheel id>, <side as int character code: 108 'l', 114 'r'>
}
```

Parsing: lines UTF-8-sanitised; blank and `//` lines skipped; the first data line names a tuneup, then skip to `{`; attribute lines split on tab, `=`, `,`, `;`, fields trimmed, attribute case-insensitive; single-value attributes need exactly 2 fields (except text fields, which need ≥ 2), `forced_wheel_side` needs exactly 3; a line exactly `}` closes the tuneup; an unclosed last tuneup is kept with a warning. A file may contain several tuneups.

Export writes exactly the fields above (tab-indented, ` = ` separators, info block, blank line, then the sets), closes with `}` and a blank line. The whole text is built in a 2000-byte buffer and truncated beyond it (with a warning if the stream wrote fewer bytes) — a rebuild should not truncate.
