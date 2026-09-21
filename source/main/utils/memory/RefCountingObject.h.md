# source/main/utils/memory/RefCountingObject.h

> Intrusive reference count shared by C++ owners and the script engine.

**Needs** — [`AppContext.h`](../../AppContext.h.md) (main-thread id) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`audio/Sound.h`](../../audio/Sound.h.md) · [`gameplay/AutoPilot.h`](../../gameplay/AutoPilot.h.md) · [`gameplay/Engine.h`](../../gameplay/Engine.h.md) · [`gameplay/VehicleAI.h`](../../gameplay/VehicleAI.h.md) · [`gui/DashBoardManager.h`](../../gui/DashBoardManager.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`physics/air/AeroEngine.h`](../../physics/air/AeroEngine.h.md) · [`physics/water/ScrewProp.h`](../../physics/water/ScrewProp.h.md) · [`resources/CacheSystem.h`](../../resources/CacheSystem.h.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.h`](../../resources/addonpart_fileformat/AddonPartFileFormat.h.md) · [`resources/tuneup_fileformat/TuneupFileFormat.h`](../../resources/tuneup_fileformat/TuneupFileFormat.h.md) · [`scripting/LocalStorage.h`](../../scripting/LocalStorage.h.md) · [`terrain/ProceduralRoad.h`](../../terrain/ProceduralRoad.h.md) · [`terrain/Terrain.h`](../../terrain/Terrain.h.md) · [`terrain/TerrainEditor.h`](../../terrain/TerrainEditor.h.md) · [`utils/GenericFileFormat.h`](../GenericFileFormat.h.md) · [`RefCountingObjectPtr.h`](RefCountingObjectPtr.h.md)
**Tier floor** — T4 in concept; exists only because the source language has no shared ownership that the script engine can see

## Purpose

Base mix-in for every object that both the game and AngelScript scripts may hold references to (actors, cache entries, terrains, engines, sound scripts…). The count lives inside the object so a raw pointer handed to the script engine and a smart pointer in C++ agree on one count.

## State

```text
RECORD RefCountingObject
  refcount : int     # starts at 0; the first owner raises it to 1
  lock     : mutex   # guards refcount; only a safety net, see Notes
```

## `AddRef`

**Contract** — increments the count. Must be called on the main thread; a debug assertion enforces this.

## `Release`

**Contract** — decrements the count; when it reaches zero the object destroys itself (as its most-derived type). Main thread only.

```text
FUNCTION release(self)
  ASSERT current thread IS main thread
  LOCK self.lock DURING
    self.refcount = self.refcount - 1
    remaining = self.refcount
  IF remaining == 0
    DESTROY self                  # after the lock is released
```

## `RegisterRefCountingObject`

**Contract** — registers type `name` with the script engine as a reference type whose add-ref/release behaviours are the two methods above.

**Notes** — the source language forces this to be a mix-in plus manual `delete this`; a T2 rebuild with a garbage collector does not need a count for C++-side lifetime at all, but it *does* need one if it embeds a native script engine that reference-counts host objects — the script engine's handles and the host's handles must keep the object alive jointly. The main-thread assertion is load-bearing as a rule: these objects are not safe to share with the physics thread by reference-count manipulation, so the physics thread holds plain references borrowed for the duration of a step.
