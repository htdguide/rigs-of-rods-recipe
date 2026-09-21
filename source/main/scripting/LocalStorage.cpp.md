# source/main/scripting/LocalStorage.cpp

> Key parsing, dirty tracking and save-on-destroy for script storage.

**Needs** — [`LocalStorage.h`](LocalStorage.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`Application.h`](../Application.h.md) · [`resources/ContentManager.h`](../resources/ContentManager.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md) · [Seam: Script engine](../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — callers of [`LocalStorage.h`](LocalStorage.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`LocalStorage.h`](LocalStorage.h.md).

## State

See header.

## Construction

Every character of the name outside `[A-Za-z0-9_-]` becomes `_` (scripts cannot escape the storage directory). Section = the given section up to its first `.`. Separator `=`. Load immediately (missing file is logged, not an error). Clean.

## Keys — `parseKey(key) → (section, key)`

`"sec.key"` addresses section `sec`; `".key"` or `"key"` address the current section. Everything after the first dot is the key.

## Typed get/set

Values are stored as text in the config format's encodings (vector3 and quaternion as space-separated numbers). Degrees are stored as radians. A missing key reads as the type's zero/empty. Every set marks dirty.

## Persistence

`saveDict` writes only when dirty; the object saves itself when its last reference goes away. Errors are logged, never raised to the script.

**Notes** — load and save always use the cache resource group, ignoring the group passed to the constructor. `deleteAll` is an empty stub. `copyFrom` also adopts the other store's filename and section, so the copy saves over the original's file.
