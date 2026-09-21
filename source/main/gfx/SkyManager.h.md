# source/main/gfx/SkyManager.h

> Caelum sky adapter (optional build): time of day, sun light, fog.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`GameContext.cpp`](../GameContext.cpp.md) · [`EnvironmentMap.cpp`](EnvironmentMap.cpp.md) · [`GfxActor.cpp`](GfxActor.cpp.md) · [`GfxScene.cpp`](GfxScene.cpp.md) · [`HydraxWater.cpp`](HydraxWater.cpp.md) · [`SkyManager.cpp`](SkyManager.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`physics/Savegame.cpp`](../physics/Savegame.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`terrain/Terrain.cpp`](../terrain/Terrain.cpp.md)
**Tier floor** — T2


## Purpose

Wraps the [Seam: Sky rendering](../../../SYSTEM-REQUIREMENTS.md#seam-sky-rendering) Caelum implementation. Compiled only with Caelum. Implementation: [`SkyManager.cpp`](SkyManager.cpp.md).

## State

```text
RECORD SkyManager = { caelum system; last clock (Julian day) }
```

## API

`LoadCaelumScript(script, fog_start = −1, fog_end = −1)`, time factor get/set, main light, pretty time "HH:MM:SS", Julian time get/set (saved in savegames as "daytime"), `NotifySkyCameraChanged(camera)`, `DetectSkyUpdate`.
