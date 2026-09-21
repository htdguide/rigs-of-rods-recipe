# source/main/gfx/EnvironmentMap.h

> A dynamic cube map around the player’s vehicle for reflective materials.

**Needs** — [`ForwardDeclarations.h`](../ForwardDeclarations.h.md)
**Used by** — [`EnvironmentMap.cpp`](EnvironmentMap.cpp.md) · [`GfxScene.h`](GfxScene.h.md)
**Tier floor** — T2


## Purpose

Vehicle materials sample a cube texture ("EnvironmentTexture") for chrome/paint reflections. Implementation: [`EnvironmentMap.cpp`](EnvironmentMap.cpp.md).

## State

```text
RECORD GfxEnvmap = { cameras[6]; render targets[6]; cube texture; update_round : 0..5 }
```

## API

`SetupEnvMap`, `UpdateEnvMap(centre, actor visuals?, full = false)`.
