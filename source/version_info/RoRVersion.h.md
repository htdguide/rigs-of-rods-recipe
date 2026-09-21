# source/version_info/RoRVersion.h

> Declares the four build-identity strings every other module reads.

**Needs** — nothing
**Used by** — [`AppContext.cpp`](../main/AppContext.cpp.md) · [`gui/OverlayWrapper.cpp`](../main/gui/OverlayWrapper.cpp.md) · [`gui/panels/GUI_CollisionsDebug.cpp`](../main/gui/panels/GUI_CollisionsDebug.cpp.md) · [`gui/panels/GUI_GameAbout.cpp`](../main/gui/panels/GUI_GameAbout.cpp.md) · [`gui/panels/GUI_GameMainMenu.cpp`](../main/gui/panels/GUI_GameMainMenu.cpp.md) · [`gui/panels/GUI_MultiplayerSelector.cpp`](../main/gui/panels/GUI_MultiplayerSelector.cpp.md) · [`gui/panels/GUI_RepositorySelector.cpp`](../main/gui/panels/GUI_RepositorySelector.cpp.md) · [`main.cpp`](../main/main.cpp.md) · [`network/CurlHelpers.cpp`](../main/network/CurlHelpers.cpp.md) · [`network/Network.cpp`](../main/network/Network.cpp.md) · [`network/OutGauge.cpp`](../main/network/OutGauge.cpp.md) · [`scripting/GameScript.cpp`](../main/scripting/GameScript.cpp.md) · [`utils/Utils.cpp`](../main/utils/Utils.cpp.md) · [`RoRVersion.cpp.in`](RoRVersion.cpp.in.md)
**Tier floor** — T4: four constant strings

## Purpose

Build identity lives in its own tiny unit so that regenerating it on every build (the date and git hash change constantly) does not force the rest of the program to recompile. Consumers only see four immutable strings.

## State

```text
CONST ROR_VERSION_STRING_SHORT : text   # "YYYY.MM", e.g. "2026.01"
CONST ROR_VERSION_STRING       : text   # short + suffix, e.g. "2026.01-dev-2cc94b1-dirty"
CONST ROR_BUILD_DATE           : text   # "YYYY-MM-DD", UTC
CONST ROR_BUILD_TIME           : text   # "HH:MM", UTC
```

## `ROR_VERSION_STRING_SHORT`

**Contract** — year and month of the release (or of the build, for dev builds). Used where a stable, comparable version is needed: the mod-cache header and savegames record it, and the About box shows it.

## `ROR_VERSION_STRING`

**Contract** — the full human-readable version including the development suffix. Sent to multiplayer servers as `clientversion` (see [`RoRnet.h`](../main/network/RoRnet.h.md) `UserInfo`, 25 bytes — the string must fit) and written to the top of the log.

## `ROR_BUILD_DATE` / `ROR_BUILD_TIME`

**Contract** — when the binary was produced, in UTC. Informational only; shown in the About box and log header.

**Notes** — a rebuild can produce these from its own build system any way it likes; see [`RoRVersion.cpp.in`](RoRVersion.cpp.in.md) for the rule that forms the suffix.
