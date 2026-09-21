# source/main/gui/panels/GUI_GameChatBox.cpp

> Two view modes over the same message view, and chat submission including /whisper.

**Needs** — [`GUI_GameChatBox.h`](GUI_GameChatBox.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`Application.h`](../../Application.h.md) · [`gameplay/ChatSystem.h`](../../gameplay/ChatSystem.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`utils/InputEngine.h`](../../utils/InputEngine.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_GameChatBox.h`](GUI_GameChatBox.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_GameChatBox.h`](GUI_GameChatBox.h.md).

## State

See header.

## `Draw`

The feed sits bottom-left, full width minus edge padding, one third of the screen tall, transparent.

- On the very first draw, post the hint "Press <key> to spawn a vehicle".
- **Feed mode** (not visible): non-interactive; notices, warnings, errors and script messages shown; each message lives 10 s (then fades); smooth scrolling.
- **History mode** (visible): the feed moves up above an input bar; only chat and general messages; lifetime 1 hour; scrolling, snapped to the bottom on opening.
- With `mp_chat_auto_hide` off, messages live a month.

Input bar (history mode): "Message" field focused on open; Escape closes; Enter sends (when connected), clears and closes.

## `SubmitMessage`

Empty → nothing. `/whisper <user> <word>` → private message to that user (unknown user → warning; missing arguments → usage notice). Anything else → broadcast.

**Notes** — the whisper parser takes only the first word after the user name as the message; a rebuild takes the rest of the line.
