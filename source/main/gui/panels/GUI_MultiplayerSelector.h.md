# source/main/gui/panels/GUI_MultiplayerSelector.h

> The multiplayer screen: online server list, direct connect, and multiplayer settings.

**Needs** — [`Application.h`](../../Application.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_MultiplayerSelector.cpp`](GUI_MultiplayerSelector.cpp.md) · [`main.cpp`](../../main.cpp.md)
**Tier floor** — T2


## Purpose

Choose a server and join. Implementation: [`GUI_MultiplayerSelector.cpp`](GUI_MultiplayerSelector.cpp.md).

## State

```text
RECORD MpServerInfo: has password, name, terrain, users "n / max", host, port, protocol version (and display forms)
RECORD MultiplayerSelector
  server list; selected row (−1); title "Multiplayer (Rigs of Rods <ver> | <proto>)"; visible; table shown; spinner shown
  buffers: user token, player name, password, host; lock icon; select-settings-tab request
  status message + colour; transport error text; HTTP status text
```

## API

`SetVisible` (first open triggers a refresh), `IsVisible`, `StartAsyncRefresh`, `Draw`, `DisplayRefreshFailed(info)`, `UpdateServerlist(list)`, `SetSettingsTabSelected`.
