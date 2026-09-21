# source/main/gui/panels/GUI_GameAbout.cpp

> Draws version info, contributor credits and the list of libraries in this build.

**Needs** — [`GUI_GameAbout.h`](GUI_GameAbout.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`Application.h`](../../Application.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`RoRVersion.h`](../../../../source/version_info/RoRVersion.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md) · [`network/RoRnet.h`](../../network/RoRnet.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_GameAbout.h`](GUI_GameAbout.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_GameAbout.h`](GUI_GameAbout.h.md).

## State

See header.

## `Draw`

475 px wide, screen height − 40, centred on first show. Shows game version, network protocol version, build date/time; then credits grouped as Authors, Current Developers, Server Contributors, Code Contributors, Core Content Contributors, Mod Contributors, Testers (names and roles are data — copy them from the original); then "Used Libs", listing optional libraries only when compiled in (sky, scripting, audio, HTTP, sockets) and the always-present ones (renderer, water, both GUIs, translation catalogs, input, vegetation paging, threads, JSON).
