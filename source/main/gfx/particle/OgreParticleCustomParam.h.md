# source/main/gfx/particle/OgreParticleCustomParam.h

> Per-particle custom data slot (a 4-vector) for shader-driven particles.

**Needs** — nothing in this repository (only standard or third-party headers)
**Used by** — [`OgreShaderParticleRenderer.cpp`](OgreShaderParticleRenderer.cpp.md)
**Tier floor** — T2


## Purpose

Visual data attached to each particle so affectors can pass a value to the [shader particle renderer](OgreShaderParticleRenderer.h.md).

## State

```text
RECORD ParticleCustomParam = { value : Vec4 = 0 }
```
