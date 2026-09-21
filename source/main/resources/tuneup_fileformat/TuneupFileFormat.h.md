# source/main/resources/tuneup_fileformat/TuneupFileFormat.h

> The vehicle tuning record: which add-on parts are fitted, what they change, and what the player forced or protected.

**Needs** — [`Application.h`](../../Application.h.md) · [`resources/CacheSystem.h`](../CacheSystem.h.md) · [`utils/memory/RefCountingObject.h`](../../utils/memory/RefCountingObject.h.md)
**Used by** — [`GameContext.cpp`](../../GameContext.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../../gui/panels/GUI_TopMenubar.cpp.md) · [`physics/Actor.cpp`](../../physics/Actor.cpp.md) · [`physics/ActorManager.cpp`](../../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../../physics/ActorSpawner.cpp.md) · [`physics/Savegame.cpp`](../../physics/Savegame.cpp.md) · [`physics/SimData.cpp`](../../physics/SimData.cpp.md) · [`resources/CacheSystem.cpp`](../CacheSystem.cpp.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.cpp`](../addonpart_fileformat/AddonPartFileFormat.cpp.md) · [`TuneupFileFormat.cpp`](TuneupFileFormat.cpp.md)
**Tier floor** — T4

## Purpose

Tuning lets a player customise a vehicle without editing its file: install **add-on parts** (which can move nodes, swap wheels/props/flexbodies/materials, or remove parts) and toggle individual elements in a menu. The whole state is one `TuneupDef`, saved as a `.tuneup` file per vehicle and applied at spawn by [`ActorSpawner`](../../physics/ActorSpawner.cpp.md). The same record type is also (ab)used to hold one add-on part's extracted tweaks for conflict detection. Implementation: [`TuneupFileFormat.cpp`](TuneupFileFormat.cpp.md).

Three layers decide the final value of any element, in priority order:
1. **forced** — the player's explicit choice in the tuning UI;
2. **tweaked / unwanted** — what installed add-on parts ask for, unless the element is **protected** by the player;
3. **original** — the vehicle file.

## State

```text
RECORD NodeTweak     { node: NodeNum, position: (x,y,z), origin: addonpart filename }
RECORD CineCamTweak  { cinecam: id, position, origin }
RECORD WheelTweak    { wheel: id, media: [text, text], side: WheelSide (INVALID = keep),
                       tire_radius: real (-1 = keep), rim_radius: real (-1 = keep), origin }
                       # media[0]: face material (wheels) or rim mesh (mesh/flexbody wheels)
                       # media[1]: band material or tyre material
RECORD PropTweak     { prop: id, media: [mesh, special mesh/beacon material], offset, rotation, origin }
RECORD FlexbodyTweak { flexbody: id, media: mesh, offset, rotation, origin }
RECORD ManagedMatTweak { name, type, media: [text ×3], origin }

RECORD TuneupDef
  # info
  name, guid (target vehicle, lowercase), filename (target vehicle), thumbnail, description,
  author_name, author_id (-1), category_id
  # add-on parts and what they contribute
  use_addonparts     : set<filename>
  node_tweaks, cinecam_tweaks, wheel_tweaks, prop_tweaks, flexbody_tweaks : map<id, tweak>
  managedmat_tweaks  : map<name, tweak>
  unwanted_props, unwanted_flexbodies, unwanted_flares, unwanted_exhausts : set<id>
  unwanted_managedmats : set<name>
  # player overrides
  force_remove_props, force_remove_flexbodies, force_remove_flares, force_remove_exhausts : set<id>
  force_remove_managedmats : set<name>
  force_wheel_sides  : map<wheel, WheelSide>
  force_video_cam_roles : map<videocamera, VideoCamRole>
  # player protections (block add-on part changes)
  protected_nodes, protected_cinecams, protected_wheels, protected_props,
  protected_flexbodies, protected_flares, protected_exhausts : set<id>
  protected_managedmats : set<name>
```

## Predicates

**Contract** — `is<X>Protected(id)`, `is<X>Unwanted(id)`, `is<X>ForceRemoved(id)` are set-membership tests; `isWheelSideForced(id, out)` and `isVideoCameraRoleForced(id, out)` return the forced value if any. `clone()` copies the record (see `.cpp` for which fields).

## `TuneupUtil`

**Contract** — static helpers every spawner step calls to get "the value to use": `getTweaked<X><Field>(tuneup, id, original)`, `is<X>Tweaked(tuneup, id, out)`, `is<X>AnyhowRemoved(tuneup, id)` (unwanted **or** force-removed), `isAddonPartUsed(tuneup, file)`, and the parse/export functions. All accept an absent tuneup and then return the original.
