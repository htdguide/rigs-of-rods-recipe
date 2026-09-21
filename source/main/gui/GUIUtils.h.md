# source/main/gui/GUIUtils.h

> Shared immediate-mode widgets: colour-marked wrapped text, settings-bound controls, input-binding labels, hold-to-confirm buttons, hyperlinks.

**Needs** — [`Application.h`](../Application.h.md) · [`GUIManager.h`](GUIManager.h.md) · [Seam: Immediate-mode GUI](../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`gfx/GfxScene.cpp`](../gfx/GfxScene.cpp.md) · [`GUIManager.cpp`](GUIManager.cpp.md) · [`GUIUtils.cpp`](GUIUtils.cpp.md) · [`gui/panels/GUI_CollisionsDebug.cpp`](panels/GUI_CollisionsDebug.cpp.md) · [`gui/panels/GUI_ConsoleView.cpp`](panels/GUI_ConsoleView.cpp.md) · [`gui/panels/GUI_FlexbodyDebug.cpp`](panels/GUI_FlexbodyDebug.cpp.md) · [`gui/panels/GUI_GameChatBox.cpp`](panels/GUI_GameChatBox.cpp.md) · [`gui/panels/GUI_GameMainMenu.cpp`](panels/GUI_GameMainMenu.cpp.md) · [`gui/panels/GUI_GameSettings.cpp`](panels/GUI_GameSettings.cpp.md) · [`gui/panels/GUI_LoadingWindow.cpp`](panels/GUI_LoadingWindow.cpp.md) · [`gui/panels/GUI_MainSelector.cpp`](panels/GUI_MainSelector.cpp.md) · [`gui/panels/GUI_MultiplayerClientList.cpp`](panels/GUI_MultiplayerClientList.cpp.md) · [`gui/panels/GUI_MultiplayerSelector.cpp`](panels/GUI_MultiplayerSelector.cpp.md) · [`gui/panels/GUI_RepositorySelector.cpp`](panels/GUI_RepositorySelector.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](panels/GUI_SurveyMap.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](panels/GUI_TopMenubar.cpp.md) · [`gui/panels/GUI_VehicleInfoTPanel.cpp`](panels/GUI_VehicleInfoTPanel.cpp.md) · [`main.cpp`](../main.cpp.md)
**Tier floor** — T2


## Purpose

Widgets several panels need that the GUI library lacks. Implementation: [`GUIUtils.cpp`](GUIUtils.cpp.md).

## State

```text
RECORD ImTextFeeder — lays out text runs onto a draw list
  draw list; origin, cursor (screen space); accumulated size; current line height
```

## API

- Text: `ImTextFeeder` (`AddInline`, `AddWrapped`, `AddMultiline`, `AddRectWrapped`, `NextLine`), `DrawColorMarkedText(drawlist, pos, default colour, alpha, wrap width, line) → size`, `StripColorMarksFromText`, `ImTextWrappedColorMarked`.
- Settings-bound controls (read the setting, write it back on change): `DrawGCheckbox`, `DrawGIntCheck`, `DrawGIntBox`, `DrawGIntSlider`, `DrawGFloatSlider`, `DrawGFloatBox`, `DrawGTextEdit`, `DrawGCombo`.
- Input bindings: `ImDrawEventHighlighted`, `ImDrawEventHighlightedButton`, `ImDrawModifierKeyHighlighted`, `ImCalcEventHighlightedSize`.
- Misc: `LoadingIndicatorCircle`, `DrawImageRotated`, `FetchIcon`, `GetImDummyFullscreenWindow`, `GetScreenPosFromWorldPos`, `ImAddItemToComboboxString`, `ImTerminateComboboxString`, `ImButtonHoldToConfirm`, `ImMoveTextInputCursorToEnd`, `ImDummyHyperlink`, `ImHyperlink`.
