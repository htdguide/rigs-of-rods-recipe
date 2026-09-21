# source/main/gfx/GfxWater.h

> The built-in water renderer: a large plane following the camera, optional reflection/refraction render targets, sea bottom plane, CPU waves.

**Needs** — [`IGfxWater.h`](IGfxWater.h.md) · [`Application.h`](../Application.h.md)
**Used by** — [`gameplay/AutoPilot.cpp`](../gameplay/AutoPilot.cpp.md) · [`gameplay/Character.cpp`](../gameplay/Character.cpp.md) · [`DustPool.cpp`](DustPool.cpp.md) · [`GfxWater.cpp`](GfxWater.cpp.md) · [`gfx/camera/CameraManager.cpp`](camera/CameraManager.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`physics/ActorForcesEuler.cpp`](../physics/ActorForcesEuler.cpp.md) · [`physics/water/Buoyance.cpp`](../physics/water/Buoyance.cpp.md) · [`physics/water/ScrewProp.cpp`](../physics/water/ScrewProp.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md)
**Tier floor** — T2


## Purpose

Default implementation of [`IGfxWater`](IGfxWater.h.md). Implementation: [`GfxWater.cpp`](GfxWater.cpp.md).

## State

```text
RECORD GfxWater
  visible, visual water height, bottom height, plane scale (1.5 for maps < 1500 m), frame counter, map size
  water plane mesh (100×100 cells, dynamic) + local vertex copy, entity, node; force-reposition flag
  reflection camera/target (+ clip plane 0.15 above), refraction camera/target (+ clip plane 0.15 below), listeners
  sea-bottom plane + node
  forced camera transform (for rendering from other viewpoints)
```
