# source/main/scripting/LocalStorage.h

> Persistent key/value store for scripts, one file per name, sectioned, typed.

**Needs** — [`Application.h`](../Application.h.md) · [`utils/ImprovedConfigFile.h`](../utils/ImprovedConfigFile.h.md) · [`utils/memory/RefCountingObject.h`](../utils/memory/RefCountingObject.h.md) · [Seam: Script engine](../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`LocalStorage.cpp`](LocalStorage.cpp.md) · [`ScriptEngine.cpp`](ScriptEngine.cpp.md) · [`scripting/bindings/LocalStorageAngelscript.cpp`](bindings/LocalStorageAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

Lets scripts keep data between sessions (race records, settings). Built on the game's sectioned config-file format ([`utils/ConfigFile`](../utils/ConfigFile.h.md)). Exposed to scripts as `LocalStorage` by [`bindings/LocalStorageAngelscript.cpp`](bindings/LocalStorageAngelscript.cpp.md). Implementation: [`LocalStorage.cpp`](LocalStorage.cpp.md).

## State

```text
RECORD LocalStorage (reference-counted)
  settings : section → multimap key → text     (inherited)
  filename : "<sanitised name>.asdata"; resource group; current section
  saved : bool                                  # false after any change ("dirty")
```

## API

Construct(name, section, group = cache) · `copyFrom(other)` · `changeSection(s)` · `get`/`set` for string, int, float, bool, vector3, quaternion, radian, degree · `saveDict` · `loadDict → ok` · `eraseKey` · `exists` · `deleteAll` · `parseKey`.
