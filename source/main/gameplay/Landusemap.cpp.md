# source/main/gameplay/Landusemap.cpp

> Parses the landuse config and rasterises the colour map into ground-model pointers.

**Needs** — [`Landusemap.h`](Landusemap.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`Application.h`](../Application.h.md) · [`physics/collision/Collisions.h`](../physics/collision/Collisions.h.md) · [`system/Console.h`](../system/Console.h.md) · [`utils/ErrorUtils.h`](../utils/ErrorUtils.h.md) · [`GameContext.h`](../GameContext.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md)
**Used by** — callers of [`Landusemap.h`](Landusemap.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`Landusemap.h`](Landusemap.h.md).

## State

See header.

## `loadConfig(file)`

INI format (tab/`:`/`=` separators), from a resource group or an absolute path; a load error goes to the console ("Error while loading landuse config").

```text
[general] or [config]
  texture = <image>                      # colour map covering the whole terrain
  frictionconfig / loadGroundModelsConfig = <file>   # extra ground models loaded into the collision system
  defaultuse = <ground model name>
[use-map]
  0xAARRGGBB = <ground model name>       # key must be exactly 10 characters, else ignored
```

Then (with the paging seam) sample the image once per metre over the terrain with no filtering, swap red/blue if the image is stored as ABGR, and store the ground model named for each colour (unknown colours → none, which the ground collision replaces with gravel). Image errors are logged and leave the map empty.
