# source/main/gui

> The user interface: two GUI toolkits, the fallback instrument HUDs, data-driven vehicle dashboards, and every window in the game.

Three presentation technologies coexist, each for a reason:

- **Immediate-mode GUI** ([seam](../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)) — every window and menu in [`panels/`](panels/README.md), plus the widgets in [`GUIUtils`](GUIUtils.h.md). It is also exposed to scripts, which ties scripts to its API.
- **Retained layout GUI** ([seam](../../../SYSTEM-REQUIREMENTS.md#seam-retained-layout-gui)) — kept only for [dashboards](DashBoardManager.h.md), because hundreds of community dashboards are authored as its layout files; dashboards can render into textures via [`RTTLayer`](RTTLayer.h.md).
- **Renderer overlays** — the built-in aircraft, boat and machine HUDs in [`OverlayWrapper`](OverlayWrapper.h.md), used when a vehicle has no dashboard.

[`GUIManager`](GUIManager.h.md) owns all of it and decides what draws in the main menu, during simulation (live data, sim thread synced) and from the sim-buffer snapshot. The vendored immediate-mode library under `gui/imgui/` is not twinned (see the root README's skip list).

## Reading order

1. [`GUIUtils`](GUIUtils.h.md) ([impl](GUIUtils.cpp.md))
2. [`RTTLayer`](RTTLayer.h.md) ([impl](RTTLayer.cpp.md))
3. [`DashBoardManager`](DashBoardManager.h.md) ([impl](DashBoardManager.cpp.md))
4. [`OverlayWrapper`](OverlayWrapper.h.md) ([impl](OverlayWrapper.cpp.md))
5. [panels/](panels/README.md)
6. [`GUIManager`](GUIManager.h.md) ([impl](GUIManager.cpp.md))

## Cycle

`GUIManager` owns every panel and panels reach back into it (theme, keyboard capture, other panels). Read panels as leaves that assume a manager exists.
