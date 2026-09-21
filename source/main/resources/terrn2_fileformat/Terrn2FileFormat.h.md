# source/main/resources/terrn2_fileformat/Terrn2FileFormat.h

> The `.terrn2` terrain descriptor: name, geometry config, environment, and lists of object, script and preset files.

**Needs** — [`ForwardDeclarations.h`](../../ForwardDeclarations.h.md) · [`physics/SimConstants.h`](../../physics/SimConstants.h.md)
**Used by** — [`GameContext.cpp`](../../GameContext.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../../gui/panels/GUI_TopMenubar.cpp.md) · [`resources/CacheSystem.cpp`](../CacheSystem.cpp.md) · [`Terrn2FileFormat.cpp`](Terrn2FileFormat.cpp.md) · [`terrain/Terrain.cpp`](../../terrain/Terrain.cpp.md) · [`terrain/TerrainGeometryManager.cpp`](../../terrain/TerrainGeometryManager.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../../terrain/TerrainObjectManager.cpp.md)
**Tier floor** — T4

## Purpose

A terrain mod's entry point. It names the heightmap configuration (`.otc`), sky and water settings, where the player spawns, and which `.tobj` object lists, `.as` scripts, asset packs and AI waypoint presets belong to the terrain. Parsing: [`Terrn2FileFormat.cpp`](Terrn2FileFormat.cpp.md).

## State

```text
RECORD Terrn2Author    { type: text, name: text }
RECORD Terrn2Telepoint { position: (x, y, z), name: text }

RECORD Terrn2Document
  name, guid                 : text
  ogre_ter_conf_filename     : text        # .otc file; empty = no heightmap terrain
  ambient_color              : colour
  category_id                : int
  start_position             : (x, y, z)
  start_rotation             : degrees; start_rotation_specified : bool
  version                    : int
  gravity                    : real        # m/s², negative = down
  has_water                  : bool; water_height, water_bottom_height : real
  caelum_config, cubemap_config, hydrax_conf_file, skyx_config : text
  caelum_fog_start, caelum_fog_end : int
  traction_map_file          : text        # land-use map for ground models
  custom_material_name       : text
  authors                    : list<Terrn2Author>
  tobj_files, as_files, assetpack_files, ai_presets_files : list<text>
  teleport_map_image         : text
  telepoints                 : list<Terrn2Telepoint>
```

## `Terrn2Parser.LoadTerrn2(stream)`

**Contract** — returns the document, or nothing when the terrain name is empty or the geometry config is not an `.otc`. See `.cpp` twin.
