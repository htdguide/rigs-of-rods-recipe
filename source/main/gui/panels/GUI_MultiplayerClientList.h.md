# source/main/gui/panels/GUI_MultiplayerClientList.h

> The player list in the top-right corner during multiplayer, with per-peer options.

**Needs** — [`network/RoRnet.h`](../../network/RoRnet.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_MultiplayerClientList.cpp`](GUI_MultiplayerClientList.cpp.md) · [`main.cpp`](../../main.cpp.md)
**Tier floor** — T2


## Purpose

Who is on the server, their rank and country, whether their vehicles loaded, and local mute/hide controls. Implementation: [`GUI_MultiplayerClientList.cpp`](GUI_MultiplayerClientList.cpp.md).

## State

```text
RECORD MpClientList
  users (local user first), peer options (parallel; local = 0) — refreshed only on a "clients refresh" message
  peer-options menu: open for list position (−1 none), corners; content width 150, margin 10, hover margin 100
  icons: stream arrows down/up × ok/grey/red, rank flags red/blue/green, warning triangle
```

## API

`Draw`, `UpdateClients`.
