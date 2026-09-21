# source/main/resources/ContentManager.cpp

> Resource-system bootstrap, mod-folder mounting for the cache scan, and content-error workarounds.

**Needs** — [`ContentManager.h`](ContentManager.h.md) · [`Application.h`](../Application.h.md) · [`gfx/ColoredTextAreaOverlayElementFactory.h`](../gfx/ColoredTextAreaOverlayElementFactory.h.md) · [`utils/ErrorUtils.h`](../utils/ErrorUtils.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md) · [`skin_fileformat/SkinFileFormat.h`](skin_fileformat/SkinFileFormat.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md) · [`CacheSystem.h`](CacheSystem.h.md) · [`gfx/particle/OgreShaderParticleRenderer.h`](../gfx/particle/OgreShaderParticleRenderer.h.md) · [`gfx/particle/FireExtinguisherAffectorFactory.h`](../gfx/particle/FireExtinguisherAffectorFactory.h.md) · [`gfx/particle/ExtinguishableFireAffectorFactory.h`](../gfx/particle/ExtinguishableFireAffectorFactory.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine) · [Seam: JSON](../../../SYSTEM-REQUIREMENTS.md#seam-json)
**Used by** — callers of [`ContentManager.h`](ContentManager.h.md) (see its Used by)
**Tier floor** — T3

## Purpose

Defines where every resource comes from and in which order it is registered — order matters because base materials must exist before any mod references them.

## State

See header twin.

## `AddResourcePack(pack, override_group = "")`

**Contract** — mounts `resources/<name>.zip` if it exists, else the folder `resources/<name>`, else raises "data not found". Without an override group, the pack's own group is used and initialised once (a second call is a no-op); with an override, the pack is mixed into that group (used to give each vehicle bundle access to shared textures/meshes/materials) and the caller initialises it.

## `InitContentManager`

Order:
1. Mount writable folders: config → `Config`, savegames → `Savegames`, scripts → `Scripts`, logs → `Logs`.
2. Register this object as the material/particle script-compiler listener.
3. Initialise **managed materials** into their global group (below).
4. Register as the resource-loading listener (if none set); default visibility of all scene objects = "visible to depth map".
5. Packs `mygui`, `dashboards`.
6. Windows only: register the shader-based particle renderer. With scripting: register the fire-extinguisher and extinguishable-fire particle affectors.
7. With audio: create the sound-script manager and load the `sounds` pack flagged as **base sounds** (never unloaded).
8. Register the coloured-text overlay element.
9. Default 5 mipmaps; texture filtering from `gfx_texture_filter`, anisotropy clamp(`gfx_anisotropy`, 1, 16).
10. Initialise all resource groups (errors logged, not fatal); end base-sound mode; create the mesh LOD generator.

## `InitManagedMaterials(group)`

**Contract** — managed materials are RoR-provided base materials (e.g. `managed/flexmesh_standard/…`) that vehicle files instantiate by name. Mount, in order: if PSSM shadows are on — `managed_materials/shadows/pssm/on/shared` (global group only) and `…/pssm/on`; otherwise `…/pssm/off`; then `managed_materials/texture`, then `managed_materials` itself. Only the global group is initialised here.

## `InitModCache(validity)`

**Contract** — builds a temporary `Content` group spanning all content locations, runs the cache check/update, and destroys the group:

```text
mount writable: cache dir -> Cache, thumbnails dir -> Thumbnails, repo attachments dir -> RepoAttachments
Content group (non-recursive):
  extra mod path (if set);
  user_dir/{mods, packs, terrains, vehicles, projects};
  program_dir/content;
  program resources beamobjects.zip, dashboards.zip, gadgets.zip
every subdirectory (recursively) of the folder locations is also added to Content, writable
IF validity == UNKNOWN: validity = cache.evaluate_cache_validity()
cache.load_mod_cache(validity)
destroy Content
```

**Notes** — subdirectories are added writable even during the scan because the engine remembers an archive's read-only flag globally and refuses to re-mount it with a different flag; project folders must be writable for tuning saves.

## `LoadGameplayResources`

**Contract** — once: packs `airfoils`, `textures`, `famicons`, `materials`, `meshes`, `overlays`, `particles`. Every call: `hydrax` if water mode is Hydrax, `caelum` or `SkyX` per sky mode, `paged` if vegetation is not NONE.

## `ListAllUserContent`

**Contract** — text listing (one per line) of every directory in the content group, followed by every file that is inside an archive or whose name matches `^.\.(airplane|boat|car|fixed|load|machine|skin|terrn2|train|truck)$` (case-insensitive). Its hash detects added/removed content.

**Notes** — the pattern allows exactly one character before the extension, so in practice only archives and directories contribute; adding a loose truck file to an existing folder is caught instead by the per-entry modification-time check or not at all. A rebuild should use `.*` as intended.

## `resourceCollision`

**Contract** — when two resources with the same name enter one group (asset packs and shared packs are mixed into mod groups), log "Skipping resource with duplicate name" and keep the **original**.

## Script-compiler guards

- a material script defining a material with an empty name → report handled without creating it, so loading of that material fails cleanly instead of corrupting state;
- a particle system template whose name already exists (templates are global, ignoring groups) → skip it instead of failing the whole group.

## JSON helpers

**Contract** — `LoadAndParseJson(file, group, out doc)` reads the resource as text and parses it (NaN/Infinity accepted); returns false on missing file (engine logs it), read error or parse error (logged). `SerializeAndWriteJson(file, group, doc)` writes compact JSON (NaN/Infinity allowed), overwriting; false on error or short write. `DeleteDiskFile(file, group)` deletes via the resource system, false on error.
