# source/main/scripting/bindings/TerrainAngelscript.cpp

> Script types `TerrainClass` and `TerrainEditorObjectClass`, enum `SpecialObjectType`.

**Needs** — [`Application.h`](../../Application.h.md) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [`terrain/Terrain.h`](../../terrain/Terrain.h.md) · [`terrain/TerrainEditor.h`](../../terrain/TerrainEditor.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `TerrainClass`

`getTerrainName`, `getTerrainFileName`, `getTerrainFileResourceGroup`, `getGUID`, `getVersion`, `getCacheEntry`, `isFlat`, `getHeightAt(x, z)`, `getSpawnPos`, `getSpawnRot`, `getMaxTerrainSize`, `addSurveyMapEntity`, `delSurveyMapEntities`, `getProceduralManager`. See [`terrain/Terrain`](../../terrain/Terrain.h.md).

## `TerrainEditorObjectClass`

A placed terrain object: `getPosition`/`setPosition`, `getRotation`/`setRotation`, `getName`, `getInstanceName`, `getType`, `get/setSpecialObjectType`, `get/setActorInstanceId`. Enum `SpecialObjectType`: NONE, TRUCK, LOAD, MACHINE, BOAT, TRUCK2. See [`terrain/TerrainEditor`](../../terrain/TerrainEditor.h.md).
