# source/main/gui/OverlayWrapper.h

> The built-in fallback HUDs (aircraft, boat, machine, tyre pressure, race timer) drawn with the rendering engine overlay system.

**Needs** — [`Application.h`](../Application.h.md) · [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine)
**Used by** — [`AppContext.cpp`](../AppContext.cpp.md) · [`Application.cpp`](../Application.cpp.md) · [`GameContext.cpp`](../GameContext.cpp.md) · [`gfx/GfxScene.cpp`](../gfx/GfxScene.cpp.md) · [`gfx/camera/CameraManager.cpp`](../gfx/camera/CameraManager.cpp.md) · [`GUIManager.cpp`](GUIManager.cpp.md) · [`OverlayWrapper.cpp`](OverlayWrapper.cpp.md) · [`main.cpp`](../main.cpp.md)
**Tier floor** — T2


## Purpose

When a vehicle ships no dashboard of its own, the game shows these fixed instrument panels. They are defined as named overlay elements and materials in the game's own resources (`tracks/...`) and driven here by rotating/scrolling needle textures. The aircraft panel is also clickable (throttles, engine starts, autopilot). Implementation: [`OverlayWrapper.cpp`](OverlayWrapper.cpp.md).

## State

```text
INTERFACE AeroInteractiveWidget: UpdateMouseHover() → hovered; GetHoveredElement()
RECORD AeroEngineWidget: throttle element (own material copy), engine-fire lamp, rpm/pitch/torque needle textures
RECORD AeroSwitchWidget: element, on/off material copies
RECORD AeroTrimWidget: up/down buttons (own material copies), hovered button, value display
RECORD AeroDashOverlay
  4 engine widgets, 4 engine-start switches; dash and needle overlays
  needle/texture handles: ADI bugs and tape, HSI rose/bug/vertical/horizontal, airspeed, altimeter, VVI, AOA; altitude text
  autopilot switches hdg, wlv, nav, alt, vs, ias, gpws, brks; trims hdg, alt, vs, ias
  throttle track top/height; drag in progress; widget list; hovered widget
RECORD OverlayWrapper
  window; dashboard visible; autopilot click cooldown
  overlays: truck pressure (+needle), aerial, marine (+needles), machine, racing
  land: gear text, auto-shift letters R N D 2 1, speedo and tacho needle textures; pressure needle texture
  marine: two throttle sliders, depth text, speed and steer needles
  race: lap time, best lap time
  loaded overlays with original scale (for aspect correction)
```

## API

`ToggleDashboardOverlays(actor)`, `showDashboardOverlays(show, actor)`, `windowResized`, `resizeOverlay`, `handleMouseMoved/Pressed/Released → consumed`, `UpdatePressureOverlay`, `update(dt)`, `UpdateLandVehicleHUD`, `UpdateAerialHUD`, `UpdateMarineHUD`, `ShowRacingOverlay`, `HideRacingOverlay`, `UpdateRacingGui`, `loadOverlay(name, auto-aspect)`.
