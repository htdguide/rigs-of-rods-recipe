# source/main/gfx/particle/OgreShaderParticleRenderer.cpp

> Vertex buffer layout and fill for shader-rendered particles.

**Needs** — [`OgreShaderParticleRenderer.h`](OgreShaderParticleRenderer.h.md) · [`OgreParticleCustomParam.h`](OgreParticleCustomParam.h.md)
**Used by** — callers of [`OgreShaderParticleRenderer.h`](OgreShaderParticleRenderer.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`OgreShaderParticleRenderer.h`](OgreShaderParticleRenderer.h.md).

## State

See header.

## Contract

Each frame: (re)allocate buffers for the particle quota (4 vertices and 6 indices per particle); for every live particle write four vertices at the particle position with the enabled attributes (colour; corner texture coordinates; size; rotation and speed; direction; time-to-live, total, fraction and inverse fraction); keep a bounding radius around the parent node; render with the particle system's material in its render queue, in world space or local space as configured, sorted per the sort mode. Parameters are exposed to particle scripts (`vertex_format_colour`, …).
