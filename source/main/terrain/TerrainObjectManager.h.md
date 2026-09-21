# source/main/terrain/TerrainObjectManager.h

> Everything placed on a terrain: .tobj files, ODEF objects, vegetation, procedural roads, predefined actors, localizers, map icons.

**Needs** — [`Application.h`](../Application.h.md) · [`resources/odef_fileformat/ODefFileFormat.h`](../resources/odef_fileformat/ODefFileFormat.h.md) · [`utils/MeshObject.h`](../utils/MeshObject.h.md) · [`ProceduralManager.h`](ProceduralManager.h.md) · [`SurveyMapEntity.h`](SurveyMapEntity.h.md)
**Used by** — [`gameplay/AutoPilot.h`](../gameplay/AutoPilot.h.md) · [`gfx/GfxScene.cpp`](../gfx/GfxScene.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](../gui/panels/GUI_SurveyMap.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`system/ConsoleCmd.cpp`](../system/ConsoleCmd.cpp.md) · [`Terrain.cpp`](Terrain.cpp.md) · [`TerrainEditor.cpp`](TerrainEditor.cpp.md) · [`TerrainObjectManager.cpp`](TerrainObjectManager.cpp.md)
**Tier floor** — T2


## Purpose

Owns the placed content of the loaded terrain and the list of editable objects used by the terrain editor and scripts. Implementation: [`TerrainObjectManager.cpp`](TerrainObjectManager.cpp.md).

## State

```text
RECORD Localizer = { type : HORIZONTAL | VERTICAL | NDB | VOR; position; rotation }   # ILS/nav beacons for the autopilot

RECORD TerrainObjectManager
  localizers : list<Localizer>
  odef_cache : map<name, ODEF document>
  tobj_cache : list<TObj document>            # kept for writing edits back
  editor_objects : list<TerrainEditorObject>  # static objects and predefined actors, in load order
  has_predefined_actors
  animated objects {entity, node, animation, speed}; particle objects; mesh objects
  map_entities : list<SurveyMapEntity>
  procedural_manager; vegetation page geometries
  scene grouping nodes (terrain / current tobj / current script) — diagnostics only
```

## API

`LoadTObjFile`, `LoadTerrainObject(odef, pos, rot°, instance, type, render distance, collisions = true, script handler = −1, uniquify material = true) → ok`, `LoadTerrainScript(file) → ok`, `moveObjectVisuals`, `destroyObject(instance)`, telepoints, predefined actors (`LoadPredefinedActors`, `SpawnSinglePredefinedActor`, `HasPredefinedActors`, `GetEditorObjectFlagRotYXZ`), `UpdateTerrainObjects(dt)`, vegetation processors, localizers, procedural manager, grouping node.
