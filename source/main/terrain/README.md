# source/main/terrain

> The loaded world: heightfield geometry, placed objects and vegetation, procedural roads, water and sky setup, and the in-game editor.

A terrain is described by a `.terrn2` file (see [`resources/terrn2_fileformat`](../resources/terrn2_fileformat/README.md)), which points to an `.otc` heightfield config, `.tobj` object lists (which reference `.odef` object definitions), scripts, and water/sky settings. [`Terrain`](Terrain.h.md) loads them in a fixed order and owns the per-terrain subsystems, including the static [collision](../physics/collision/README.md) world.

## Reading order

1. [`SurveyMapEntity.h`](SurveyMapEntity.h.md)
2. [`TerrainGeometryManager`](TerrainGeometryManager.h.md) (+ the third-party [`OgreTerrainPSSMMaterialGenerator`](OgreTerrainPSSMMaterialGenerator.h.md))
3. [`ProceduralRoad`](ProceduralRoad.h.md) → [`ProceduralManager`](ProceduralManager.h.md)
4. [`TerrainEditor`](TerrainEditor.h.md) (the editable object record) → [`TerrainObjectManager`](TerrainObjectManager.h.md)
5. [`Terrain`](Terrain.h.md)
