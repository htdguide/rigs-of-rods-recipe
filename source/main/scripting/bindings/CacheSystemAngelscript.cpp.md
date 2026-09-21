# source/main/scripting/bindings/CacheSystemAngelscript.cpp

> Script access to the installed-content cache.

**Needs** — [`physics/Actor.h`](../../physics/Actor.h.md) · [`AngelScriptBindings.h`](AngelScriptBindings.h.md) · [`resources/CacheSystem.h`](../../resources/CacheSystem.h.md) · [`ScriptEngine.h`](../ScriptEngine.h.md) · [`ScriptUtils.h`](../ScriptUtils.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `CacheEntryClass`

Read-only properties: fpath, fname, fext, dname (display name), categoryid, categoryname, resource_bundle_type, resource_bundle_path, number, deleted, filecachename, resource_group.

## `CacheSystemClass` (global `modcache`)

`findEntryByFilename(type, partial, name)`, `getEntryByNumber`, `query(dictionary) → dictionary` (filter by type, category, search text; returns result count and entries). Enum `LoaderType`: NONE, TERRAIN, VEHICLE, TRUCK, CAR, BOAT, AIRPLANE, TRAILER, TRAIN, LOAD, EXTENSION, SKIN, ALLBEAM, ADDONPART, TUNEUP, ASSETPACK, DASHBOARD, GADGET. See [`resources/CacheSystem`](../../resources/CacheSystem.h.md).
