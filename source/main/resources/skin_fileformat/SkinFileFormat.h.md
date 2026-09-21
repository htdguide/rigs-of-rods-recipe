# source/main/resources/skin_fileformat/SkinFileFormat.h

> The `.skin` document: texture and material substitutions that restyle a vehicle.

**Needs** — [`Application.h`](../../Application.h.md) · [Seam: 3D rendering engine](../../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine) (data stream)
**Used by** — [`physics/ActorSpawner.cpp`](../../physics/ActorSpawner.cpp.md) · [`resources/CacheSystem.cpp`](../CacheSystem.cpp.md) · [`resources/ContentManager.cpp`](../ContentManager.cpp.md) · [`SkinFileFormat.cpp`](SkinFileFormat.cpp.md)
**Tier floor** — T4

## Purpose

A skin mod swaps textures or whole materials on a vehicle without editing it. One `.skin` file may contain several skins. Parsing: [`SkinFileFormat.cpp`](SkinFileFormat.cpp.md).

## State

```text
RECORD SkinDocument
  name              : text
  guid              : text      # lowercase; the vehicle GUID this skin applies to
  thumbnail         : text      # preview image file name
  description       : text
  author_name       : text
  author_id         : int = -1
  replace_textures  : map<original texture name, replacement texture name>
  replace_materials : map<original material name, replacement material name>
```

## `SkinParser.ParseSkins(stream)`

**Contract** — returns every skin in the file, in order. See `.cpp` twin.
