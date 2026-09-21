# source/main/gui/panels/GUI_ConsoleWindow.h

> The developer console window: message log, command line with history, and menus for commands, script examples and the script monitor.

**Needs** — [`Application.h`](../../Application.h.md) · [`GUI_ConsoleView.h`](GUI_ConsoleView.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui) · [`GUI_AngelScriptExamples.h`](GUI_AngelScriptExamples.h.md) · [`GUI_ScriptMonitor.h`](GUI_ScriptMonitor.h.md)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_ConsoleWindow.cpp`](GUI_ConsoleWindow.cpp.md)
**Tier floor** — T2


## Purpose

Interactive front end to [`system/Console`](../../system/Console.h.md). Implementation: [`GUI_ConsoleWindow.cpp`](GUI_ConsoleWindow.cpp.md).

## State

```text
CONSTANT history cap 100
RECORD ConsoleWindow
  visible, hovered; script examples; script monitor; message view (scrolling enabled)
  command buffer (500 chars); command history; history cursor (−1 = editing a new line)
```

## API

`SetVisible`, `IsVisible`, `IsHovered`, `Draw`, `doCommand(text)`.
