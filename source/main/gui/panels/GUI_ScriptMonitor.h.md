# source/main/gui/panels/GUI_ScriptMonitor.h

> Console tab listing running scripts and recently used ones, with reload/stop/autoload controls.

**Needs** — [`Application.h`](../../Application.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`GUI_ConsoleWindow.h`](GUI_ConsoleWindow.h.md) · [`GUI_ScriptMonitor.cpp`](GUI_ScriptMonitor.cpp.md)
**Tier floor** — T2


## Purpose

Script lifecycle control for modders. Implementation: [`GUI_ScriptMonitor.cpp`](GUI_ScriptMonitor.cpp.md).

## State

```text
RECORD ScriptMonitor: recent display list (recent scripts that are not running)
```

## API

`Draw`.
