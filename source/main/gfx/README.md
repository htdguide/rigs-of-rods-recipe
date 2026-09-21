# source/main/gfx

> Everything drawn: per-actor visuals, the scene update, water, sky, shadows, particles, skidmarks, cameras.

Graphics never drives physics. Once per frame, while the simulation thread is synchronised, [`GfxScene::BufferSimulationData`](GfxScene.cpp.md) copies what visuals need into [sim buffers](SimBuffers.h.md); the simulation then resumes and [`GfxScene::UpdateScene`](GfxScene.cpp.md) updates visuals from those copies, farming mesh deformation out to worker threads. The rendering library itself is the [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine); water and sky renderers are pluggable seams (the vendored Hydrax and SkyX libraries under `hydrax/` and `skyx/` are not twinned).

## Reading order

1. [`SimBuffers.h`](SimBuffers.h.md), [`GfxData.h`](GfxData.h.md)
2. [`DustPool`](DustPool.h.md), [`Skidmark`](Skidmark.h.md), [`particle/`](particle/README.md)
3. [`GfxActor`](GfxActor.h.md) → [`GfxScene`](GfxScene.h.md)
4. [`camera/`](camera/README.md)
5. Environment: [`EnvironmentMap`](EnvironmentMap.h.md), [`ShadowManager`](ShadowManager.h.md), [`IGfxWater`](IGfxWater.h.md) → [`GfxWater`](GfxWater.h.md), [`HydraxWater`](HydraxWater.h.md), [`SkyManager`](SkyManager.h.md), [`SkyXManager`](SkyXManager.h.md), [`SurveyMapTextureCreator`](SurveyMapTextureCreator.h.md)
6. Text and utilities: [`MovableText`](MovableText.h.md), [`ColoredTextAreaOverlayElement`](ColoredTextAreaOverlayElement.h.md), [`AdvancedScreen.h`](AdvancedScreen.h.md)
