# source/main/gui/panels/GUI_GameChatBox.h

> The on-screen chat: a fading message feed, expandable into a scrollable history with an input line.

**Needs** — [`Application.h`](../../Application.h.md) · [`GUI_ConsoleView.h`](GUI_ConsoleView.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_GameChatBox.cpp`](GUI_GameChatBox.cpp.md)
**Tier floor** — T2


## Purpose

Shows recent messages over the game and lets the player chat in multiplayer. Implementation: [`GUI_GameChatBox.cpp`](GUI_GameChatBox.cpp.md).

## State

```text
RECORD GameChatBox
  visible (false = feed only, true = history + input); keyboard focused; input buffer (400 chars)
  message view: actor-area messages off, command replies off, errors on, icons on, padding 2×1
  first-draw hint pending; scroll initialised; move caret to end pending
```

## API

`SetVisible`, `IsVisible`, `Draw`, `GetConsoleView`, `AssignBuffer(text)` (prefill, caret to end), `SubmitMessage`.
