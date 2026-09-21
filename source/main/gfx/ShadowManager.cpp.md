# source/main/gfx/ShadowManager.cpp

> PSSM parameters per quality level and shared split points for materials.

**Needs** — [`ShadowManager.h`](ShadowManager.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`camera/CameraManager.h`](camera/CameraManager.h.md) · [`GfxScene.h`](GfxScene.h.md)
**Used by** — callers of [`ShadowManager.h`](ShadowManager.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`ShadowManager.h`](ShadowManager.h.md).

## State

See header.

## Behaviour

Shadow colour (1.063, 1.078, 1.125). With PSSM: additive integrated texture shadows, directional extrusion 299 m, far distance 350 m, 3 shadow textures (32-bit float), self-shadowing, back-face casters, depth caster material. Texture sizes and split lambda by quality: ultra 4096/3072/2048, λ 0.965; high 3072/2048/2048, 0.97; medium 2048/1024/1024, 0.975; low 1024/1024/512, 0.98. Three split points from the camera near clip to the shadow far distance with λ, padding = near clip; the split points are published to the shared shader parameter `pssmSplitPoints` so all materials agree. The terrain material receives depth PSSM shadows (no low-LOD shadows, no light map).
