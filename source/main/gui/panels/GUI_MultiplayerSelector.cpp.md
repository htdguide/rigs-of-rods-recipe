# source/main/gui/panels/GUI_MultiplayerSelector.cpp

> Fetches the server list from the portal on a worker thread and joins servers.

**Needs** — [`GUI_MultiplayerSelector.h`](GUI_MultiplayerSelector.h.md) · [`Application.h`](../../Application.h.md) · [`resources/ContentManager.h`](../../resources/ContentManager.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [`network/RoRnet.h`](../../network/RoRnet.h.md) · [`RoRVersion.h`](../../../../source/version_info/RoRVersion.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_MultiplayerSelector.h`](GUI_MultiplayerSelector.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_MultiplayerSelector.h`](GUI_MultiplayerSelector.h.md).

## State

See header.

## Server list fetch (worker thread)

`GET <mp_api_url>/server-list?json=true` ([Seam: HTTP client](../../../../SYSTEM-REQUIREMENTS.md#seam-http-client), [Seam: JSON](../../../../SYSTEM-REQUIREMENTS.md#seam-json)). Transport error or non-200 → post REFRESH_SERVERLIST_FAILURE with title, transport code, HTTP code. The body must be a JSON array of objects with `name`, `terrain-name`, `ip`, `port`, `has-password`, `current-users`, `max-clients`, `version` (e.g. `RoRnet_2.45`); otherwise failure "Server returned invalid data". Success posts the parsed list. The main thread applies either message.

**Notes** — on invalid JSON the original posts the failure with a text payload instead of the failure-info record the handler expects; a rebuild posts the same record type for both failures.

## Tabs

- **Online (click to refresh)** — spinner while loading; then a table: name (+ lock icon if passworded), terrain, users, version (green when equal to ours, red otherwise), host:port. Double-click or *Join* (only for compatible servers) stores password/host/port and requests connect; a password field appears for passworded servers. Messages: error (with transport and HTTP details) or "There are no available servers".
- **Direct IP** — host, port, password, *Join*.
- **Settings** — auto connect, auto-hide chat, hide net labels, hide own label, multiplayer collisions, include remote vehicles when cycling, nickname, default password, user token (+ link to get one; warning never to share it).
