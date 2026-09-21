# source/main/resources — content formats and the mod database

Chapter 7. Depends on the hub, `utils/` (tokeniser, config files, hashing) and `system/`; used by physics (spawning), terrain, gfx, GUI and scripting.

Two halves:
- **File formats** — one sub-chapter per format, each a pure parser/serializer producing a document: the [truck format](rig_def_fileformat/README.md) (the big one), [skins](skin_fileformat/SkinFileFormat.h.md), [add-on parts](addonpart_fileformat/AddonPartFileFormat.h.md), [tune-ups](tuneup_fileformat/TuneupFileFormat.h.md), and the terrain family [`.terrn2`](terrn2_fileformat/Terrn2FileFormat.h.md) → [`.otc`](otc_fileformat/OTCFileFormat.h.md) (heightmap) + [`.tobj`](tobj_fileformat/TObjFileFormat.h.md) (placements) → [`.odef`](odef_fileformat/ODefFileFormat.h.md) (objects).
- **Content management** — [`ContentManager`](ContentManager.h.md) mounts built-in packs and user folders into resource groups; [`CacheSystem`](CacheSystem.h.md) scans mod bundles into `mods.cache`, answers queries, loads bundles on demand, and manages user projects and tune-up saves.

| Twin | Role |
|---|---|
| [`rig_def_fileformat/`](rig_def_fileformat/README.md) | Truck file format (sub-chapter) |
| [`skin_fileformat/SkinFileFormat.h`](skin_fileformat/SkinFileFormat.h.md) · [`.cpp`](skin_fileformat/SkinFileFormat.cpp.md) | `.skin` |
| [`terrn2_fileformat/Terrn2FileFormat.h`](terrn2_fileformat/Terrn2FileFormat.h.md) · [`.cpp`](terrn2_fileformat/Terrn2FileFormat.cpp.md) | `.terrn2` |
| [`otc_fileformat/OTCFileFormat.h`](otc_fileformat/OTCFileFormat.h.md) · [`.cpp`](otc_fileformat/OTCFileFormat.cpp.md) | `.otc` master + pages |
| [`odef_fileformat/ODefFileFormat.h`](odef_fileformat/ODefFileFormat.h.md) · [`.cpp`](odef_fileformat/ODefFileFormat.cpp.md) | `.odef` |
| [`tobj_fileformat/TObjFileFormat.h`](tobj_fileformat/TObjFileFormat.h.md) · [`.cpp`](tobj_fileformat/TObjFileFormat.cpp.md) | `.tobj` + legacy road import |
| [`tuneup_fileformat/TuneupFileFormat.h`](tuneup_fileformat/TuneupFileFormat.h.md) · [`.cpp`](tuneup_fileformat/TuneupFileFormat.cpp.md) | Tuning record + `.tuneup` |
| [`addonpart_fileformat/AddonPartFileFormat.h`](addonpart_fileformat/AddonPartFileFormat.h.md) · [`.cpp`](addonpart_fileformat/AddonPartFileFormat.cpp.md) | `.addonpart`, merge & conflicts |
| [`CacheSystem.h`](CacheSystem.h.md) · [`.cpp`](CacheSystem.cpp.md) | Mod database |
| [`ContentManager.h`](ContentManager.h.md) · [`.cpp`](ContentManager.cpp.md) | Resource-system setup |
