# source/main/gui/panels/GUI_ConsoleView.h

> Reusable renderer for console messages, used by both the console window and the in-game chat box.

**Needs** — [`Application.h`](../../Application.h.md) · [`system/Console.h`](../../system/Console.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`GUI_AngelScriptExamples.h`](GUI_AngelScriptExamples.h.md) · [`GUI_ConsoleView.cpp`](GUI_ConsoleView.cpp.md) · [`GUI_ConsoleWindow.h`](GUI_ConsoleWindow.h.md) · [`GUI_GameChatBox.h`](GUI_GameChatBox.h.md)
**Tier floor** — T2


## Purpose

One message renderer with two personalities: scrolling log (console) or bottom-anchored fading feed (chat). Implementation: [`GUI_ConsoleView.cpp`](GUI_ConsoleView.cpp.md).

## State

```text
RECORD ConsoleView
  filters by level: notice, warning, error, chat, commands (all on)
  filters by area: log echo (off), script, actor, terrain (on); general info always passes
  smooth scrolling (on); message lifetime ms (0 = forever); scrolling enabled (off; also splits multi-line messages);
  icons (off); background colour and padding; line spacing 1; alpha; fade-out interval 700 ms
  filtered messages (copies), display list (rebuilt each frame), reload requested, messages consumed so far
```

## API

`DrawConsoleMessages`, `DrawFilteringOptions`, `RequestReloadMessages`.
