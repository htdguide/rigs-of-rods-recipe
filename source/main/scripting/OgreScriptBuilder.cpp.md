# source/main/scripting/OgreScriptBuilder.cpp

> Loads a script section by file name from any resource group, preferring a _DEVEL variant.

**Needs** — [`OgreScriptBuilder.h`](OgreScriptBuilder.h.md) · [`Application.h`](../Application.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`system/Console.h`](../system/Console.h.md) · [`resources/ContentManager.h`](../resources/ContentManager.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [Seam: Script engine](../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — callers of [`OgreScriptBuilder.h`](OgreScriptBuilder.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`OgreScriptBuilder.h`](OgreScriptBuilder.h.md).

## State

See header.

## Loading a section (also called for each #include)

```text
filename = part after the last "/"                     # include paths are flattened: resources are found by name
IF .as AND diagnostic setting "load devel scripts": try "<base>_DEVEL.as" in any group (notice on success)
ELSE / not found: open filename in any group            (missing → console error, −2)
hash = SHA-1(source)
hand the source to the preprocessor under the name actually opened
```

**Notes** — the hash is overwritten by every included file, so it identifies the last file processed, not the whole program; it is only used as `script-hash` in online API submissions. A rebuild that wants a program identity hashes the concatenation.
