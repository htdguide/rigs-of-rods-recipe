# source/main/scripting/OgreScriptBuilder.h

> Script builder that resolves source files and #includes through the game resource system.

**Needs** — [`Application.h`](../Application.h.md) · [Seam: Script engine](../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`OgreScriptBuilder.cpp`](OgreScriptBuilder.cpp.md) · [`ScriptEngine.cpp`](ScriptEngine.cpp.md)
**Tier floor** — T2


## Purpose

The script library's stock builder reads files from disk and supports `#include`; this variant reads them from the virtual resource system instead (so scripts inside zipped mods work), and records a content hash. Implementation: [`OgreScriptBuilder.cpp`](OgreScriptBuilder.cpp.md).

## State

```text
RECORD OgreScriptBuilder: hash : text   # SHA-1 of the most recently loaded section
```

## `GetHash() → text`

See implementation.
