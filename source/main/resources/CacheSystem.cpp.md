# source/main/resources/CacheSystem.cpp

> Cache validity, scanning bundles, extracting metadata per file type, JSON persistence, querying, bundle loading, and project (user copy / tune-up save) management.

**Needs** — [`CacheSystem.h`](CacheSystem.h.md) · [`ContentManager.h`](ContentManager.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`addonpart_fileformat/AddonPartFileFormat.h`](addonpart_fileformat/AddonPartFileFormat.h.md) · [`rig_def_fileformat/RigDef_Parser.h`](rig_def_fileformat/RigDef_Parser.h.md) · [`skin_fileformat/SkinFileFormat.h`](skin_fileformat/SkinFileFormat.h.md) · [`terrn2_fileformat/Terrn2FileFormat.h`](terrn2_fileformat/Terrn2FileFormat.h.md) · [`tuneup_fileformat/TuneupFileFormat.h`](tuneup_fileformat/TuneupFileFormat.h.md) · [`utils/GenericFileFormat.h`](../utils/GenericFileFormat.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`gui/panels/GUI_LoadingWindow.h`](../gui/panels/GUI_LoadingWindow.h.md) · [`gui/panels/GUI_GameMainMenu.h`](../gui/panels/GUI_GameMainMenu.h.md) · [`gfx/GfxActor.h`](../gfx/GfxActor.h.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [Seam: JSON](../../../SYSTEM-REQUIREMENTS.md#seam-json) · [Seam: Zip archives](../../../SYSTEM-REQUIREMENTS.md#seam-zip-archives)
**Used by** — callers of [`CacheSystem.h`](CacheSystem.h.md) (see its Used by)
**Tier floor** — T3

## Purpose

Everything about discovering installed content and turning a cache entry into loaded, usable resources.

## State

See header twin.

## Validity check (at startup)

```text
FUNCTION evaluate_cache_validity()      # called by ContentManager while the content group exists
  hash_generated = fast_hash(content_manager.list_all_user_content())
  IF load_cache_file() != VALID: RETURN (its result)       # missing/corrupt/wrong format -> NEEDS_REBUILD
  IF hash_loaded != hash_generated: RETURN NEEDS_UPDATE     # bundles added or removed
  FOR EACH entry
    path = bundle path (+ "/" + fname for folder bundles)
    IF modification_time(path) != entry.filetime: RETURN NEEDS_UPDATE
  RETURN VALID
```

## `LoadModCache(validity)`

```text
FUNCTION load_mod_cache(validity)
  update_time = now
  IF validity != VALID
    IF NEEDS_REBUILD: clear_cache()           # delete mods.cache, unload groups, delete preview images
    ELSE:             unload all bundle groups; prune_cache()
    with console echo of the log temporarily off:
      parse_zip_archives(content group)       # every *.zip and *.skinzip
      parse_known_files(content group)        # loose files in content folders
    detect_duplicates()
    write_cache_file(); load_cache_file()     # re-read so numbering is fresh
  loaded = true
```

**Pruning** — entries whose bundle/file is missing or whose modification time changed are flagged deleted (and their preview images removed); surviving paths are remembered so they are not re-scanned.

**Scanning an archive** — each archive not yet known is mounted alone in a temporary group, all files with known extensions inside it are added, and the group is destroyed. A progress window ("Loading zips in group …") is shown; at the end the main menu is told the cache updated.

## `AddFile(group, file, ext)` — metadata extraction

**Contract** — skips files already present (same name and bundle, not deleted). Otherwise opens the file and creates one or more entries:

| Extension | Metadata source |
|---|---|
| `terrn2` | [terrn2 parser](terrn2_fileformat/Terrn2FileFormat.cpp.md): name, category, guid → uniqueid, version, authors |
| `skin` | one entry per skin in the file: name, guid, description, author |
| `addonpart` | generic tokens: `addonpart_name`, `addonpart_description`, `addonpart_guid` (repeatable → set), `addonpart_filename` (repeatable → set); all lowercased where ids |
| `tuneup` | one entry per tuneup: name, guid, description, category, author, target filename |
| `assetpack` | `assetpack_name`, `assetpack_description`, `assetpack_author type id name email` |
| `dashboard` | `dashboard_name`, `dashboard_description`, `dashboard_category`, `dashboard_author …` |
| `gadget` | `gadget_name`, `gadget_description` (repeatable, joined with newlines), `gadget_category`, `gadget_author …` |
| truck family | parsed with the [truck parser](rig_def_fileformat/RigDef_Parser.cpp.md) (legacy importer disabled): see below |

Then for each new entry: guid lowercased; `fname`, `fname_without_uid` (see below), extension, path; `filetime` of the archive (zip bundles) or of the file (folder bundles); bundle type/path; `number = count + 1`; `addtimestamp = update_time`; preview image extracted; the entry appended; and a "mod cache activity: entry added" script event fired (which also retries pending multiplayer spawns).

**Truck summary** — display name = title line (or `@<filename>` if empty); description lines joined; authors (forum id only if given); default skin = last `default_skin`; module names; last engine in the root module gives gear count, min/max rpm, torque, engine type (from last `engoption`, default `t`); last `fileinfo` gives uniqueid, category, version (else `"-1"`, −1, −1); vehicle type: TRUCK if any engine, else BOAT if screwprops, else AIRPLANE if turbojets/pistonprops/turboprops — checked over user modules first, then the root module overrides; last `globals` gives masses; counts of nodes, beams, shocks(+shocks2), fixes, hydros, commands, flares, props, wings, turboprops, rotators(+2), exhausts, turbojets, flexbodies, sound sources (**counted twice** from the same list — original bug); wheel count over all five wheel sections and "propelled" wheels; if `axles` exist, propelled wheels = 2 × axle count.

**Preview images** — for skins, the skin's `preview` file; otherwise `<basename>-mini.dds|png|jpg` next to the mod file (first that exists). Copied into the cache group as `<bundlename>_<fname>.mini.<ext>` (skins: `<bundlename>_<previewbase>.mini.<ext>`).

## Name helpers

- `StripUIDfromString(s)` — if `s` contains `-` preceded immediately by `UID`, drop everything up to and including that `-` (legacy repository prefixes like `abcUID-truck.truck`).
- `StripSHA1fromString(s)` — if the first `-` or `_` is at position ≥ 20, drop the prefix up to it (repository SHA-1 prefixes on archive names).

## Duplicate detection

**Contract** — two non-deleted entries are duplicates when their UID-less file names, trimmed lowercase display names, and bundle base names (lowercased, spaces and hyphens → `_`, SHA-1 prefix stripped) all match. If they are in the same bundle, the one with the longer internal path is kept and the shorter-path one flagged deleted; if in different bundles, both are kept and logged as "possible duplicate".

## Cache file (`mods.cache`)

JSON object `{ "format_version": 14, "global_hash": "<hex>", "entries": [ … ] }`. Each entry object has: `usagecounter`, `addtimestamp`, `resource_bundle_type`, `resource_bundle_path`, `fpath`, `fname`, `fname_without_uid`, `fext`, `filetime`, `dname`, `categoryid`, `uniqueid`, `guid`, `version`, `filecachename`, `authors` [{`type`,`name`,`email`,`id`}], `description`, `tags`, `default_skin`, `fileformatversion`, `hasSubmeshs`, the counts, `truckmass`, `loadmass`, `minrpm`, `maxrpm`, `torque`, `customtach`, `custom_particles`, `forwardcommands`, `importcommands`, `rescuer`, `driveable` (int), `numgears`, `enginetype` (int char code), `sectionconfigs` [..], `addonpart_guids` [..], `addonpart_filenames` [..], `tuneup_associated_filename`. Deleted entries are not written. On load, an unknown category or one ≥ 9000 becomes Unsorted; entries are renumbered from 1. Internal format — a rebuild may change it freely (bump the version).

## `FindEntryByFilename(type, partial, name)`

```text
FUNCTION find(type, partial, name)                # name may be "bundle.zip:file.truck"
  (bundle, file) = split at first ':'; lowercase both
  best_partial = none (shortest matching fname wins)
  FOR EACH entry
    skip unless (type is Terrain) == (entry is terrn2)
             AND (type is DashBoard) == (entry is dashboard)
             AND NOT (type is AllBeam AND entry is skin)
    IF lower(fname) == file OR lower(fname_without_uid) == file
      IF bundle empty OR lower(bundle basename) == bundle: RETURN entry
      ELSE remember as candidate
    ELSE IF partial AND lower(fname) contains file AND shorter than best
      IF bundle matches: best_partial = entry ELSE remember as candidate
  IF candidates: print "Mod '<name>' was not found in cache; candidates (N) are:" and each "bundle:file"
  RETURN best_partial if partial else none
```

## `Query(query)`

```text
FUNCTION query(q)
  lowercase q.search, q.guid, q.target_filename
  FOR EACH entry
    IF q.guid set: addon parts must list it in addonpart_guids; others must equal it
    IF q.target_filename set: addon parts with a non-empty filename set must contain it;
                              tuneups with a target filename must equal it
    type filter (by extension):
      terrn2->Terrain; skin->Skin; addonpart->AddonPart; tuneup->Tuneup; assetpack->AssetPack;
      dashboard->DashBoard; gadget->Gadget;
      truck -> AllBeam|Vehicle|Truck;  car -> AllBeam|Vehicle|Truck|Car;
      boat -> AllBeam|Boat; airplane -> AllBeam|Airplane; train -> AllBeam|Train;
      trailer -> AllBeam|Trailer|Extension; load -> AllBeam|Load|Extension
      (machine and fixed are never listed)
    count category usage (entry's category, All, and Fresh if added < 1 day ago)
    category filter: a real category (≤ 9000) must match; Fresh requires fresh; other pseudo-categories pass
    search → (match, score):
      FULLTEXT: display name (+0), filename (+100), description (+200), author names (+300), emails (+400)
      GUID: guid; AUTHORS: names/emails; WHEELS: "<wheels>x<propelled>"; FILENAME: filename (+100)
      NONE: match all, score 0
      score = base + position of the substring
    IF match: add result; track newest addtimestamp
  sort results by score, ties by lowercase display name
```

## Bundle loading

**Resource group naming** — `{bundle USER:<path under user dir>}`, `{bundle BIN:<path under program dir>}`, `{bundle EXTRA:<path under extra mod path>}`, else `{bundle FULL:<absolute path>}` (prefix match case-insensitive). Stable names let saves and scripts refer to bundles.

```text
FUNCTION load_resource(entry)
  IF entry.resource_group set: load_supplementary(entry); RETURN
  group = compose name
  read_only = bundle is a zip OR bundle is a content-dir root folder   # project folders must be writable
  CASE entry.fext
    terrn2:   group in the global pool (vegetation paging requires it)
    skin, tuneup: private group + managed materials
    gadget:   private group + managed materials + built-in script includes
    else (vehicles, add-on parts, asset packs…): private group + managed materials
              + built-in textures, materials and meshes packs
  initialise group; entry.resource_group = group; load_supplementary(entry)
  mark every entry sharing this bundle path as loaded into the same group
  on failure: log, destroy the half-made group
```

`load_supplementary` parses the `.skin` or `.tuneup` file once and attaches each contained definition to all entries of the same file whose display name matches. `UnLoadResource` clears cached documents of every entry in that group, marks them unloaded and destroys the group (callers must despawn actors first); `ReLoadResource` = unload + load.

`LoadAssetPack(target, file)` mounts an asset pack's bundle **into the target's resource group** (read-only unless a folder) and re-initialises that group; name collisions keep the original (see [`ContentManager`](ContentManager.cpp.md#resourcecollision)). Missing pack → console warning.

## Projects

`CreateProject(request)` makes `sys_projects_dir/<name>/` (error if it exists and overwrite is off):
- **SAVE_TUNEUP** — writes `<name>.tuneup` with a clone of the actor's working tuneup, target guid/filename of the source vehicle, thumbnail = source preview, category 8001 (Tuneups); with overwrite, reuses the existing tuneup entry.
- **DEFAULT** / **ACTOR_PROJECT** — copies every file of the source bundle into the folder (progress window); ACTOR_PROJECT skips all truck-family files and then writes the source truck file re-tokenised with the title replaced by the project name and `fileinfo`'s category set to 8000 (Projects).
The new entry (category 8000 or 8001, folder bundle) is added, its bundle reloaded, and a "mod cache activity" event fired (added or modified).

`ModifyProject(request)` applies one change to the actor's **working tuneup** (creating it if needed) — add/remove add-on part (checks it exists and runs the conflict check first), force/unforce removals, wheel sides, video-camera roles, protections; load a saved tuneup (the working tuneup becomes a clone of the save); reset (drop the working tuneup); or write the live actor's node/beam edits back into its truck file (folder projects only). Then it **respawns** the actor: delete it, and chain a spawn at the same XZ position with the lowest node height, same heading, configuration, skin, tuneup and debug view.

`DeleteProject(entry)` unloads the bundle, deletes every file and the folder, removes the entry, and forces the tuning menu to refresh.

## Repository helpers

`IsRepoFileInstalled(file, out path)` — true if any entry's bundle base name equals the file name. `DeleteResourceBundleByFilename(file)` — flags all entries of that bundle deleted (firing "entry deleted" events), removes them from memory, unloads the archive and deletes it from disk (firing "bundle deleted").
