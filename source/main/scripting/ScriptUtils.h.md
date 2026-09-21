# source/main/scripting/ScriptUtils.h

> Conversions between native containers and script arrays/dictionaries, plus read-only script views of native maps and lists.

**Needs** — [`ScriptEngine.h`](ScriptEngine.h.md) · [Seam: Script engine](../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`GameScript.cpp`](GameScript.cpp.md) · [`ScriptEngine.cpp`](ScriptEngine.cpp.md) · [`scripting/bindings/ActorAngelscript.cpp`](bindings/ActorAngelscript.cpp.md) · [`scripting/bindings/CacheSystemAngelscript.cpp`](bindings/CacheSystemAngelscript.cpp.md) · [`scripting/bindings/OgreAngelscript.cpp`](bindings/OgreAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

Shared glue for the bindings and the `game` API. Header-only. Compiled only with scripting support.

## State

Stateless (the views hold a reference to a native container and a reference count).

## `VectorToScriptArray(list, element type) → array`

New script array `array<T>` of the same length, element-wise copy. `MapToScriptArray(map, T)` copies the map's values in key order. `IterableMapToScriptArray(begin, end, T)` appends each value.

**Notes** — `IterableListToScriptArray` in the original appends the address of the iterator rather than the element; it is unusable as written. A rebuild appends the element.

## `GetValueFromScriptDict(context, dict, required, key, type, out) → found`

Missing dictionary, missing key, or a value whose script type differs from `type` → false; each is logged as an error only when `required`. Otherwise copies the value out. Script dictionaries store every integer as 64-bit and every float as double, so callers ask for `int64` / `double`.

## `ReadonlyScriptDictView<T>` / `ReadonlyScriptArrayView<T>`

Script-visible, reference-counted, read-only windows onto a native string-keyed map or list — no copy. Dict: `exists(key)`, `isEmpty()`, `getSize()`, `getKeys()`, `[key]` (missing → default value). Array: `isEmpty()`, `length()`, `[i]` (out of range → error). Each type registers itself under a caller-chosen script type name.

**Notes** — the view must not outlive the native container; the bindings only hand them out for containers owned by long-lived objects.

## `ScriptRefCast<A,B>` / `ScriptRefCastNoCount<A,B>`

Down-cast for script handle conversions: none → none; failed cast → none; success → the object (with an added reference for counted types).
