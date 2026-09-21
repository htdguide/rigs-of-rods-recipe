# source/main/utils/ErrorUtils.h

> Native, pre-GUI message boxes for fatal startup errors.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`AppContext.cpp`](../AppContext.cpp.md) · [`gameplay/Landusemap.cpp`](../gameplay/Landusemap.cpp.md) · [`gui/OverlayWrapper.cpp`](../gui/OverlayWrapper.cpp.md) · [`main.cpp`](../main.cpp.md) · [`network/Network.cpp`](../network/Network.cpp.md) · [`physics/collision/Collisions.cpp`](../physics/collision/Collisions.cpp.md) · [`resources/ContentManager.cpp`](../resources/ContentManager.cpp.md) · [`system/AppCommandLine.cpp`](../system/AppCommandLine.cpp.md) · [`system/AppConfig.cpp`](../system/AppConfig.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md) · [`ErrorUtils.cpp`](ErrorUtils.cpp.md)
**Tier floor** — T4

## Purpose

Startup can fail before the renderer or GUI exist (no resources folder, broken plugins). These functions show a message using only the operating system. Implementation: [`ErrorUtils.cpp`](ErrorUtils.cpp.md).

## State

Stateless.

## `ShowError(title, message)` · `ShowInfo(title, message)` · `ShowMsgBox(title, message, type)`

**Contract** — show a modal OS message box (type 0 = error, 1 = info); return 0. See the `.cpp` twin for the text wrapping `ShowError` applies.
