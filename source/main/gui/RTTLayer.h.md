# source/main/gui/RTTLayer.h

> A GUI layer that draws into a texture instead of the screen, and a pool that recycles such layers.

**Needs** —  · [Seam: Retained layout GUI](../../../SYSTEM-REQUIREMENTS.md#seam-retained-layout-gui)
**Used by** — [`DashBoardManager.cpp`](DashBoardManager.cpp.md) · [`DashBoardManager.h`](DashBoardManager.h.md) · [`GUIManager.cpp`](GUIManager.cpp.md) · [`GUIManager.h`](GUIManager.h.md) · [`RTTLayer.cpp`](RTTLayer.cpp.md) · [`physics/ActorSpawner.h`](../physics/ActorSpawner.h.md)
**Tier floor** — T2


## Purpose

Lets a dashboard layout render onto a texture that a 3D cockpit material samples. Implementation: [`RTTLayer.cpp`](RTTLayer.cpp.md).

## State

```text
RECORD RTTLayer (a GUI layer kind "RTTLayer")
  texture or none; texture size; texture name; out-of-date flag
RECORD RTTLayerManager
  next layer number (from 1); reusable layers : stack
```

## API

`setTextureSize`, `setTextureName` (both before creating), `createRttTexture`, `destroyRttTexture`, `getTextureName`; manager `CreateOrReuseRttLayer`, `RecycleRttLayer`.
