# source/main/resources/CacheSystem.h

> The mod database: one entry per installed vehicle, terrain, skin, add-on part, tune-up, asset pack, dashboard or gadget, persisted to `mods.cache`.

**Needs** — [`Application.h`](../Application.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`utils/memory/RefCountingObject.h`](../utils/memory/RefCountingObject.h.md) · [`utils/memory/RefCountingObjectPtr.h`](../utils/memory/RefCountingObjectPtr.h.md) · [`rig_def_fileformat/RigDef_File.h`](rig_def_fileformat/RigDef_File.h.md) · [`physics/SimData.h`](../physics/SimData.h.md) · [Seam: JSON](../../../SYSTEM-REQUIREMENTS.md#seam-json)
**Used by** — [`Application.cpp`](../Application.cpp.md) · [`GameContext.cpp`](../GameContext.cpp.md) · [`GameContext.h`](../GameContext.h.md) · [`gui/DashBoardManager.cpp`](../gui/DashBoardManager.cpp.md) · [`gui/panels/GUI_GameSettings.cpp`](../gui/panels/GUI_GameSettings.cpp.md) · [`gui/panels/GUI_MainSelector.cpp`](../gui/panels/GUI_MainSelector.cpp.md) · [`gui/panels/GUI_MainSelector.h`](../gui/panels/GUI_MainSelector.h.md) · [`gui/panels/GUI_TopMenubar.h`](../gui/panels/GUI_TopMenubar.h.md) · [`main.cpp`](../main.cpp.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/ActorExport.cpp`](../physics/ActorExport.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/ActorSpawnerFlow.cpp`](../physics/ActorSpawnerFlow.cpp.md) · [`physics/Savegame.cpp`](../physics/Savegame.cpp.md) · [`physics/SimData.cpp`](../physics/SimData.cpp.md) · [`physics/flex/FlexFactory.cpp`](../physics/flex/FlexFactory.cpp.md) · [`CacheSystem.cpp`](CacheSystem.cpp.md) · [`ContentManager.cpp`](ContentManager.cpp.md) · [`ContentManager.h`](ContentManager.h.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.cpp`](addonpart_fileformat/AddonPartFileFormat.cpp.md) · [`resources/rig_def_fileformat/RigDef_File.cpp`](rig_def_fileformat/RigDef_File.cpp.md) · [`resources/tuneup_fileformat/TuneupFileFormat.cpp`](tuneup_fileformat/TuneupFileFormat.cpp.md) · [`resources/tuneup_fileformat/TuneupFileFormat.h`](tuneup_fileformat/TuneupFileFormat.h.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`scripting/bindings/CacheSystemAngelscript.cpp`](../scripting/bindings/CacheSystemAngelscript.cpp.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md)
**Tier floor** — T3

## Purpose

Players install thousands of mods (zip archives or folders). Scanning and parsing all of them on every start would take minutes, so their metadata is extracted once into a cache that the selector UI, scripts, multiplayer and the tuning system query. Each archive or folder is a **resource bundle**; several entries may share one bundle, which is loaded into its own resource group only when first needed. Implementation: [`CacheSystem.cpp`](CacheSystem.cpp.md).

## State

```text
CONST CACHE_FILE = "mods.cache"         # JSON, in sys_cache_dir
CONST CACHE_FILE_FORMAT = 14            # bump to force rebuild
CONST CACHE_FILE_FRESHNESS = 86400 s    # entries newer than a day are "fresh"

RECORD AuthorInfo { id: int = -1, type, name, email }

RECORD CacheEntry
  # identity
  number              : int             # 1-based, assigned at load; not stable across sessions
  fname, fname_without_uid, fext, fpath : text
  dname               : text            # display name
  guid                : text (lowercase)
  uniqueid            : text; version : int
  categoryid          : int; categoryname : text
  # bundle
  resource_bundle_type: "Zip" | "FileSystem"
  resource_bundle_path: text            # archive file or folder
  resource_group      : text            # empty until loaded
  filetime            : seconds         # of the archive (zip) or the file (folder)
  addtimestamp        : seconds         # when added to the cache
  deleted             : bool
  usagecounter        : int
  filecachename       : text            # cached preview image in sys_cache_dir
  authors             : list<AuthorInfo>
  # lazily attached documents
  actor_def           : truck Document (after first spawn)
  skin_def, tuneup_def, terrn2_def
  addonpart_data_only : TuneupDef       # for conflict checks
  # add-on part targeting
  addonpart_guids     : set<text>       # vehicles the part fits
  addonpart_filenames : set<text>       # optional narrower filter
  tuneup_associated_filename : text     # lowercase
  # vehicle summary (for the selector)
  description, tags, default_skin : text
  fileformatversion : int; hasSubmeshs : bool
  nodecount, beamcount, shockcount, fixescount, hydroscount, wheelcount, propwheelcount,
  commandscount, flarescount, propscount, wingscount, turbopropscount, turbojetcount,
  rotatorscount, exhaustscount, flexbodiescount, soundsourcescount : int
  truckmass, loadmass, minrpm, maxrpm, torque : real
  customtach, custom_particles, forwardcommands, importcommands, rescuer : bool
  driveable : NOT_DRIVEABLE | TRUCK | AIRPLANE | BOAT | MACHINE | AI
  numgears : int; enginetype : char ('t' default)
  sectionconfigs : list<text>           # module names

ENUM CacheSearchMethod = NONE | FULLTEXT | GUID | AUTHORS | WHEELS | FILENAME   # all case-insensitive

RECORD CacheQuery
  filter_type : LoaderType; filter_category : int = All
  filter_guid, filter_target_filename : text (exact, case-insensitive; empty = off)
  search_method, search_string
  results : list<(entry, score)>        # sorted by score, then display name
  category_usage : map<category, count> # ignores search and category filter
  last_update : seconds

ENUM CacheValidity = UNKNOWN | VALID | NEEDS_UPDATE | NEEDS_REBUILD

RECORD CreateProjectRequest { name, description, source_entry, source_actor, type: DEFAULT | SAVE_TUNEUP | ACTOR_PROJECT, overwrite }
RECORD ModifyProjectRequest { target_actor, type, subject (name), subject_id, value_int }
  # types: use/unuse add-on part; force-remove / un-force prop, flexbody, flare, exhaust, managed material;
  #        force / unforce wheel side and video-camera role; protect / unprotect prop, flexbody, wheel,
  #        flare, exhaust, managed material; load tuneup save; reset tuneup; write live actor back to its file

RECORD CacheSystem
  entries           : list<CacheEntry>
  known_extensions  : machine fixed terrn2 truck car boat airplane trailer load train skin addonpart tuneup assetpack dashboard gadget
  content_dirs      : mods packs terrains vehicles projects        # under sys_user_dir
  categories        : map<id, localized name>                     # see table below
  resource_paths    : set<text>                                   # bundles already scanned this update
  hash_loaded, hash_generated : text
  update_time       : seconds
  loaded            : bool
```

## Categories

Ids come from the online repository and must not change: 108 Other Land Vehicles, 146 Street Cars, 147 Light Racing Cars, 148 Offroad Cars, 149 Fantasy Cars, 150 Bikes, 155 Crawlers, 152 Towercranes, 153 Mobile Cranes, 154 Other cranes, 107 Buses, 151 Tractors, 156 Forklifts, 159 Fantasy Trucks, 160 Transport Trucks, 161 Racing Trucks, 162 Offroad Trucks, 110 Boats, 113 Helicopters, 114 Aircraft, 117 Trailers, 118 Other Loads, 129 Addon Terrains, 859 Container, 875 Submarine, 200–203 Dashboards (Generic/Truck/Boat/RTT), 300–302 Gadgets (Generic/Actor/Terrain); local only: 5000 Official Terrains, 5001 Night Terrains, 8000 Projects, 8001 Tuneups, 9990 Unsorted, 9991 All, 9992 Fresh, 9993 Hidden.

## Operations

Startup `LoadModCache`, `IsModCacheLoaded`; lookups `FindEntryByFilename`, `GetEntryByNumber`, `FetchSkinByName`, `Query`, `IsRepoFileInstalled`; bundle loading `LoadResource`, `ReLoadResource`, `UnLoadResource`, `LoadSupplementaryDocuments`, `LoadAssetPack`; projects `CreateProject`, `ModifyProject`, `DeleteProject`; misc `GetEntries`, `GetCategories`, `GetPrettyName`, `ActorTypeToName`, `GetContentDirs`, `DeleteResourceBundleByFilename`, `ParseSingleZip`. See `.cpp`.
