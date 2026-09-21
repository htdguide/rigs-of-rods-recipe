# source/main/gfx/Skidmark.h

> Tyre marks: config lookup of mark textures by ground and slip, and per-wheel ribbon meshes.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`GfxScene.h`](GfxScene.h.md) · [`Skidmark.cpp`](Skidmark.cpp.md) · [`main.cpp`](../main.cpp.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/Savegame.cpp`](../physics/Savegame.cpp.md)
**Tier floor** — T2


## Purpose

Every wheel gets a skidmark object at spawn (even when marks are disabled). When enabled, the wheel step reports contact points and slip; marks are drawn as triangle-strip ribbons. Implementation: [`Skidmark.cpp`](Skidmark.cpp.md).

## State

```text
RECORD SkidmarkConfig = { models : map<model, list<{ground model name, texture, slip_from, slip_to}>> }   # from skidmarks.cfg
RECORD Skidmark
  wheel, config, scene node
  segments : queue<{ strip object, material, points[length], face sizes[length], textures[length], last axle point, write pos }>
  length = 500 (even) points per segment, at most 20 segments, min spacing 0.25 m, max gap max(0.5, 1.1·wheel width)
```

## API

Config: `LoadDefaultSkidmarkDefs`, `getTexture(model, ground, slip) → 0 found | 1 no model | 2 no match`. Mark: `update(contact, index, slip, ground)`, `reset`.
