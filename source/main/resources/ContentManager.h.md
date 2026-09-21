# source/main/resources/ContentManager.h

> Sets up the resource system: built-in resource packs, user directories, managed materials, the mod-cache scan, and JSON helpers.

**Needs** — [`CacheSystem.h`](CacheSystem.h.md) · [`Application.h`](../Application.h.md) · [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine) · [Seam: JSON](../../../SYSTEM-REQUIREMENTS.md#seam-json)
**Used by** — [`Application.cpp`](../Application.cpp.md) · [`gfx/Skidmark.cpp`](../gfx/Skidmark.cpp.md) · [`gui/GUIManager.cpp`](../gui/GUIManager.cpp.md) · [`gui/panels/GUI_MainSelector.cpp`](../gui/panels/GUI_MainSelector.cpp.md) · [`gui/panels/GUI_MultiplayerSelector.cpp`](../gui/panels/GUI_MultiplayerSelector.cpp.md) · [`gui/panels/GUI_RepositorySelector.cpp`](../gui/panels/GUI_RepositorySelector.cpp.md) · [`gui/panels/GUI_ScriptMonitor.cpp`](../gui/panels/GUI_ScriptMonitor.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](../gui/panels/GUI_SurveyMap.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`main.cpp`](../main.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/Savegame.cpp`](../physics/Savegame.cpp.md) · [`CacheSystem.cpp`](CacheSystem.cpp.md) · [`ContentManager.cpp`](ContentManager.cpp.md) · [`scripting/LocalStorage.cpp`](../scripting/LocalStorage.cpp.md) · [`scripting/OgreScriptBuilder.cpp`](../scripting/OgreScriptBuilder.cpp.md) · [`system/AppConfig.cpp`](../system/AppConfig.cpp.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md) · [`terrain/TerrainEditor.cpp`](../terrain/TerrainEditor.cpp.md) · [`terrain/TerrainGeometryManager.cpp`](../terrain/TerrainGeometryManager.cpp.md)
**Tier floor** — T3

## Purpose

RoR's media come from three places: built-in **resource packs** in the program's `resources/` folder, the **user directories** (config, savegames, scripts, logs, cache), and **mod bundles**. This object wires all of them into the rendering engine's resource groups and guards against malformed content. Implementation: [`ContentManager.cpp`](ContentManager.cpp.md).

## State

```text
RECORD ResourcePack { name: text, group: text }   # folder or zip "resources/<name>[.zip]"

BUILT-IN PACKS (name -> group):
  OgreCore->OgreCoreRG, wallpapers->Wallpapers, airfoils->AirfoilsRG, caelum->CaelumRG,
  cubemaps->CubemapsRG, dashboards->DashboardsRG, famicons->FamiconsRG, flags->FlagsRG,
  fonts->FontsRG, hydrax->HydraxRG, icons->IconsRG, materials->MaterialsRG, meshes->MeshesRG,
  mygui->MyGuiRG, overlays->OverlaysRG, paged->PagedRG, particles->ParticlesRG, pssm->PssmRG,
  rtshader->RtShaderRG, scripts->ScriptsRG, sounds->SoundsRG, textures->TexturesRG, SkyX->SkyXRG
  (BEAM_OBJECTS is declared but not defined)

RECORD ContentManager
  base_resources_loaded : bool
```

## Operations

`AddResourcePack(pack, override_group)`, `InitManagedMaterials(group)`, `InitContentManager()`, `InitModCache(validity)`, `LoadGameplayResources()`, `ListAllUserContent()`, `DeleteDiskFile(file, group)`, `LoadAndParseJson(file, group, out doc)`, `SerializeAndWriteJson(file, group, doc)`; engine callbacks for resource loading, collisions and script compilation. See `.cpp`.
