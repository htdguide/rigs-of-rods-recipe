# source/main/scripting/bindings/LocalStorageAngelscript.cpp

> Script type `LocalStorageClass` with a factory.

**Needs** — [`LocalStorage.h`](../LocalStorage.h.md) · [`AngelScriptBindings.h`](AngelScriptBindings.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `LocalStorageClass`

Reference-counted; created by scripts with `LocalStorage(name, section = "common")`. Methods: `copyFrom`, `changeSection`, `get`/`getString`, `set`/`setString`, typed get/set for float, vector3, radian, degree, quaternion, bool, int (also `getInteger`/`setInteger`), `save`, `reload`, `exists`, `delete` (erase key). Index access `storage["key"]` reads/writes strings. See [`LocalStorage`](../LocalStorage.h.md).
