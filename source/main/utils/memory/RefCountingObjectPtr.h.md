# source/main/utils/memory/RefCountingObjectPtr.h

> Smart handle to a `RefCountingObject`, usable from both host code and scripts.

**Needs** — [`RefCountingObject.h`](RefCountingObject.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ForwardDeclarations.h`](../../ForwardDeclarations.h.md) · [`audio/SoundScriptManager.h`](../../audio/SoundScriptManager.h.md) · [`gameplay/AutoPilot.h`](../../gameplay/AutoPilot.h.md) · [`gui/DashBoardManager.h`](../../gui/DashBoardManager.h.md) · [`physics/air/AeroEngine.h`](../../physics/air/AeroEngine.h.md) · [`physics/water/ScrewProp.h`](../../physics/water/ScrewProp.h.md) · [`resources/CacheSystem.h`](../../resources/CacheSystem.h.md) · [`utils/GenericFileFormat.h`](../GenericFileFormat.h.md)
**Tier floor** — T4 in concept

## Purpose

The owning handle type. C++ code uses it like a shared pointer; it is also registered with the script engine as a *value type that behaves like a handle*, so a script variable of type `BeamClassPtr@`-style handle (e.g. `ActorClassPtr`) and a C++ `ActorPtr` are the same thing.

## State

```text
RECORD RefCountingObjectPtr<T>
  ref : optional<T>    # the referenced object, or empty
```

Invariant: while `ref` is set, this handle contributes exactly one to `ref.refcount`.

## Construct / copy / assign / destroy

**Contract** — constructing from an object or copying another handle adds one reference; destroying or reassigning releases the previous one. Self-assignment to the same object is a no-op (checked before release, so the object is never freed and re-acquired).

```text
FUNCTION set(self, new_ref)
  IF self.ref IS new_ref: RETURN
  IF self.ref IS SET: self.ref.release()
  self.ref = new_ref
  IF self.ref IS SET: self.ref.add_ref()
```

## Comparison and access

**Contract** — equality against another handle, against a raw object, or against "empty"; dereference to the object; an identity conversion to an integer so handles can key ordered maps and be tested for emptiness.

## `RegisterRefCountingObjectPtr`

**Contract** — registers `handle_name` (e.g. `ActorPtr`) as a script value type that is also a handle to `obj_name` (e.g. `BeamClass`) and participates in the script garbage collector.

Registered surface:
- constructors: default (empty), from an object handle, from another handle; destructor
- `opImplCast()` and `getHandle()` → the object, adding a reference for the script's use
- `opHndlAssign` from a handle or from an object
- `opEquals` against a handle or an object
- GC behaviours: *enumerate references* reports the held object; *release references* empties the handle (breaks cycles)

**Notes** — the script engine's own handle syntax (`obj@`) and this wrapper coexist; scripts can pass either. When the constructor or cast is driven by the script engine, the reference is added manually because the script engine hands over a borrowed pointer, not an owned one — getting this asymmetric add-ref wrong leaks or double-frees objects on script unload.
