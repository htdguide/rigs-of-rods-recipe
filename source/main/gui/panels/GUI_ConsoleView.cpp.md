# source/main/gui/panels/GUI_ConsoleView.cpp

> Incremental message filtering, virtualised scrolling, per-player colouring and fade-out.

**Needs** — [`GUI_ConsoleView.h`](GUI_ConsoleView.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`Application.h`](../../Application.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`network/Network.h`](../../network/Network.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_ConsoleView.h`](GUI_ConsoleView.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_ConsoleView.h`](GUI_ConsoleView.h.md).

## State

See header.

## Message intake — `UpdateMessages`

Under the console's lock, take only messages added since last time (a shrinking log or a filter change triggers a full reload). Keep those passing the area and level filters; in scrolling mode split multi-line messages into trimmed non-empty lines. Returns the number added.

## Display list

Drop expired messages (when a lifetime is set). In feed mode, consecutive general-info messages that begin with the same first word replace each other, so progress lines ("Loading… 10 %", "Loading… 20 %") occupy one line; chat is never merged.

## Drawing

Two layers per message: a rounded black background at half the message alpha, then the text.

- **Feed mode** — draw from the bottom of the window upward until full.
- **Scroll mode** — reserve height for all lines, stick to the bottom when new messages arrive and the user was already at the bottom, draw only the visible slice (+2 lines), offset by the fractional line for smooth scrolling; horizontal scroll follows the widest line.

Per message: an icon (explicit, or by kind: script, notice, warning, error, chat) when icons are on; for network messages the text becomes `#RRGGBB<name>: #000000<text>` with the player's colour (also for players who have left); base colour by type (title → highlight, error, warning, reply → success, help). Rendered with colour marks ([`GUIUtils`](../GUIUtils.cpp.md#colour-marked-text)).

**Notes** — fade-out in the original is broken: `alpha` is shared across messages and decremented cumulatively, and the overtime subtraction is unsigned. A rebuild computes per message `alpha = clamp(1 − (now − (t + lifetime − fade)) / fade, 0, 1)`.
