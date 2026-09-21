# source/main/gfx/particle/OgreShaderParticleRenderer.h

> A particle renderer ("shader") that emits one quad per particle with selectable per-vertex attributes for vertex shaders.

**Needs** — nothing in this repository (only standard or third-party headers)
**Used by** — [`OgreShaderParticleRenderer.cpp`](OgreShaderParticleRenderer.cpp.md) · [`resources/ContentManager.cpp`](../../resources/ContentManager.cpp.md)
**Tier floor** — T2


## Purpose

Third-party-style extension of the renderer's particle system (a community renderer carried in-tree): rather than building billboards on the CPU, it writes particle attributes into vertices so a vertex shader can expand, rotate and fade them. Part of the [Seam: 3D rendering engine](../../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine). Implementation: [`OgreShaderParticleRenderer.cpp`](OgreShaderParticleRenderer.cpp.md).

## State

```text
RECORD ShaderParticleRenderer (type "shader")
  material, vertex/index data, render queue
  vertex format switches: colour, texture, size, rotation, rotation speed, direction, ttl, total ttl, time fragment, inverse time fragment
  texture-coordinate table (4 corners), default particle size, parent node, sort mode, radius, local-space flag
```

Plus a factory registering the "shader" renderer type.
