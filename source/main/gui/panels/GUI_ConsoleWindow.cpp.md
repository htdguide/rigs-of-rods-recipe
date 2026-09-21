# source/main/gui/panels/GUI_ConsoleWindow.cpp

> Draws the console and runs typed commands with shell-like history.

**Needs** — [`GUI_ConsoleWindow.h`](GUI_ConsoleWindow.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUI_AngelScriptExamples.h`](GUI_AngelScriptExamples.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_ConsoleWindow.h`](GUI_ConsoleWindow.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_ConsoleWindow.h`](GUI_ConsoleWindow.h.md).

## State

See header.

## `Draw`

First shown centred at 1/1.6 × 1/1.3 of the screen. Menu bar: *Filter options* (from the view); *Commands* — every registered console command with usage and description, clicking runs it with no arguments; with scripting, *AngelScript* ([examples](GUI_AngelScriptExamples.h.md)) and *Script Monitor* ([monitor](GUI_ScriptMonitor.h.md)). Body: the message view, then a one-line command input submitted with Enter. Keyboard goes to the GUI while hovered.

## `doCommand(text)`

Trim; ignore empty; append to history (dropping the oldest beyond 100); reset the cursor; hand to the console.

## History keys

Up: from a new line jump to the newest entry, else move older (stop at oldest). Down: move newer; past the newest returns to an empty new line. The input is replaced whenever the cursor moves.
