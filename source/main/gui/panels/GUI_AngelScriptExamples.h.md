# source/main/gui/panels/GUI_AngelScriptExamples.h

> A table of one-click script snippets that exercise the vehicle API, shown inside the console window.

**Needs** — [`Application.h`](../../Application.h.md) · [`GUI_ConsoleView.h`](GUI_ConsoleView.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`GUI_AngelScriptExamples.cpp`](GUI_AngelScriptExamples.cpp.md) · [`GUI_ConsoleWindow.cpp`](GUI_ConsoleWindow.cpp.md) · [`GUI_ConsoleWindow.h`](GUI_ConsoleWindow.h.md)
**Tier floor** — T2


## Purpose

Discoverability for modders: each row names an API call, offers an argument widget, and runs it on click. Implementation: [`GUI_AngelScriptExamples.cpp`](GUI_AngelScriptExamples.cpp.md).

## State

```text
RECORD AngelScriptExamples — the argument widgets' values
  scale 1.0, mass 1000, reset keep-position false, locked false, light 1, blink 1, node 1, visible false, custom light 1
```

## API

`Draw` (three columns: call, argument, description); row helpers for slider, plain, checkbox, int, int+checkbox, node rows; `ExecuteString(code)`.
