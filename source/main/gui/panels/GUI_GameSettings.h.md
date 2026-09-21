# source/main/gui/panels/GUI_GameSettings.h

> The settings window: every user-facing setting grouped into tabs, each widget bound directly to its setting.

**Needs** — [`Application.h`](../../Application.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_GameSettings.cpp`](GUI_GameSettings.cpp.md)
**Tier floor** — T2


## Purpose

Front end for the settings registry ([`system/CVar`](../../system/CVar.h.md)); settings persist through the config file. Implementation: [`GUI_GameSettings.cpp`](GUI_GameSettings.cpp.md).

## State

```text
RECORD GameSettings
  visible; window size; "must restart" banner shown; pending height bump for the banner
  dashboard category being chosen (truck | boat | none); last resolved default truck/boat dashboards
  text edit buffers: preset terrain/vehicle/config, extra mod path, OutGauge IP, default and forced EFX presets
  cached combo strings for each enum setting and the UI preset
```

## API

`Draw`, `IsVisible`, `SetVisible` (builds combo strings on first open; closing in the main menu reopens it).
