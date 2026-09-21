# source/main/gui/panels/GUI_MultiplayerClientList.cpp

> Draws player rows with health and rank icons, tooltips, and a peer-options popup.

**Needs** — [`GUI_MultiplayerClientList.h`](GUI_MultiplayerClientList.h.md) · [`Application.h`](../../Application.h.md) · [`physics/ActorManager.h`](../../physics/ActorManager.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`network/Network.h`](../../network/Network.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_MultiplayerClientList.h`](GUI_MultiplayerClientList.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_MultiplayerClientList.h`](GUI_MultiplayerClientList.h.md).

## State

See header.

## `UpdateClients`

Copy users and peer options from the network (one lock each), put the local user first, and keep the options menu attached to the same user id if it was open.

## `Draw`

Top-right, 225 px content, semi-transparent. Per user:

- a `<` button (not for yourself) toggling the peer-options menu;
- in simulation, for other players: stream health arrows — down = their vehicles as loaded here, up = ours as loaded there — plain (ok), grey (idle), red (load errors);
- rank flag: admin red, moderator blue, ranked green;
- country flag from the user's `language` field `ll_CC`;
- the name in the player's colour.

Hovering a row shows a tooltip with name, uid, language, country (+ flag), rank (long form), and the two stream states in words. When the server reports degraded network quality, a red "Slow Network Download" line is added.

**Notes** — the original stores the "down" state into the "up" icon and then overwrites it, so the down arrow never shows; a rebuild draws both.

## Peer-options menu

Placed left of the row. Local actions: *Mute chat*, *Mute actors*, *Hide actors* (checkboxes posting add/remove peer-option messages, then a list refresh). Server commands open the chat box prefilled: *Report* → `!report <uid> Please enter reason: `; admins/moderators also get *Kick* → `!kick <uid>` and *Ban* → `!ban <uid>` (commands interpreted by the server). The menu closes when the mouse leaves its box plus 100 px.
