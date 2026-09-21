# source/main/scripting/bindings/ConsoleAngelscript.cpp

> Script types `ConsoleClass` and `CVarClass`, enum `CVarFlags`.

**Needs** — [`system/Console.h`](../../system/Console.h.md) · [`AngelScriptBindings.h`](AngelScriptBindings.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `ConsoleClass` (global `console`)

`cVarCreate(name, long name, flags, value)`, `cVarFind`, `cVarGet(name, flags)` (find or create), `cVarSet`, `cVarAssign`. Scripts can create their own settings, which then persist like built-in ones when flagged ARCHIVE.

## `CVarClass`

`getName`, `getStr`, `getInt`, `getFloat`, `getBool`. Enum `CVarFlags`: CVAR_TYPE_BOOL, CVAR_TYPE_INT, CVAR_TYPE_FLOAT, CVAR_ARCHIVE, CVAR_NO_LOG. See [`system/Console`](../../system/Console.h.md), [`system/CVar`](../../system/CVar.h.md).
