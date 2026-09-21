# source/main/gui/panels

> One file pair per window or overlay. Each panel is a plain object owned by the [GUI manager](../GUIManager.h.md), drawn every frame when visible, and acts on the game by posting messages.

Panels never change simulation structures directly when a message exists for it (spawn, delete, load, connect…); they post it and the game loop applies it at a safe point. The exceptions are the tools drawn while the simulation is synced (node/beam utility, collisions debug, flexbody debug, top menu bar), which edit live objects. Most panels are written against the [immediate-mode GUI seam](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui).

## Reading order

**Building blocks** — [`ConsoleView`](GUI_ConsoleView.h.md) (message renderer), [`MessageBox`](GUI_MessageBox.h.md), [`LoadingWindow`](GUI_LoadingWindow.h.md).

**Menus and setup** — [`GameMainMenu`](GUI_GameMainMenu.h.md), [`GameSettings`](GUI_GameSettings.h.md), [`GameControls`](GUI_GameControls.h.md), [`GameAbout`](GUI_GameAbout.h.md), [`MainSelector`](GUI_MainSelector.h.md), [`RepositorySelector`](GUI_RepositorySelector.h.md).

**Multiplayer** — [`MultiplayerSelector`](GUI_MultiplayerSelector.h.md), [`MultiplayerClientList`](GUI_MultiplayerClientList.h.md), [`GameChatBox`](GUI_GameChatBox.h.md).

**In simulation** — [`TopMenubar`](GUI_TopMenubar.h.md), [`VehicleInfoTPanel`](GUI_VehicleInfoTPanel.h.md), [`SurveyMap`](GUI_SurveyMap.h.md), [`DirectionArrow`](GUI_DirectionArrow.h.md), [`SimPerfStats`](GUI_SimPerfStats.h.md).

**Developer tools** — [`ConsoleWindow`](GUI_ConsoleWindow.h.md) (with [`AngelScriptExamples`](GUI_AngelScriptExamples.h.md) and [`ScriptMonitor`](GUI_ScriptMonitor.h.md)), [`FrictionSettings`](GUI_FrictionSettings.h.md), [`NodeBeamUtils`](GUI_NodeBeamUtils.h.md), [`CollisionsDebug`](GUI_CollisionsDebug.h.md), [`FlexbodyDebug`](GUI_FlexbodyDebug.h.md), [`TextureToolWindow`](GUI_TextureToolWindow.h.md).
