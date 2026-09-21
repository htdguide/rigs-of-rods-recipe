# source/main/gui/GUIUtils.cpp

> Implementations of the shared GUI helpers; the colour-mark format is shared with multiplayer chat.

**Needs** — [`GUIUtils.h`](GUIUtils.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md) · [Seam: Immediate-mode GUI](../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUIUtils.h`](GUIUtils.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUIUtils.h`](GUIUtils.h.md).

## State

See header (plus one hold-to-confirm tracker, below).

## Colour-marked text

Text may embed `#RRGGBB` marks (hex, case-insensitive); each switches the colour of the text after it. The format is used by chat and player names coming from the network, so it is part of the multiplayer contract.

```text
colour = default
FOR each segment between marks:
  draw segment with colour (alpha overridden), wrapping at wrap width
  next mark: #000000 → back to default; a colour darker than the theme threshold in all channels → inverted (255 − c)
  so black-ish marks stay readable on the dark UI
return total size
```

`StripColorMarksFromText` removes every mark.

### Layout (`ImTextFeeder`)

`AddMultiline` splits into words and blanks (UTF-8 aware, `\r` ignored, `\n` forces a new line). `AddWrapped` moves a word to the next line when it would cross the wrap width, and drops leading blanks at the start of a wrapped line. `AddRectWrapped` places an inline box (icons) the same way.

## Settings-bound controls

Each reads the current setting, shows the widget, and writes back only on change. Int and float boxes commit on Enter. The text edit keeps an edit buffer, commits on Enter, shows "(hit Enter key to submit)" while focused, and resyncs from the setting while not focused. Float slider shows 2 decimals, float box 3.

## Input-binding labels

Show an event's trimmed key binding in a small frame; highlighted in the theme colour while the event is active. Button variant returns clicks (and optional hovered/active state). Modifier-key variant does the same for Ctrl/Shift/Alt.

## `ImButtonHoldToConfirm(id, small, seconds) → confirmed`

While the button is held, a tooltip shows "Hold to confirm" and a draining bar; returns true once when the time runs out. Releasing early resets. Only one such button can be in progress at a time (the tracker is global).

## Misc

- `LoadingIndicatorCircle` — N dots on a circle; each dot's size and colour pulse with `max(0, sin(t·speed − i·step))`.
- `DrawImageRotated` — textured quad rotated about its centre.
- `FetchIcon(name)` — texture from the icon group, loaded on demand; none on failure.
- `GetImDummyFullscreenWindow` — a transparent, input-less, full-screen window whose draw list panels use for world-space labels.
- `GetScreenPosFromWorldPos` — camera projection to GUI pixels; true only for points in front of the camera.
- Combo strings — items separated by NUL, list terminated by two NULs (the GUI library's combo format).
- `ImMoveTextInputCursorToEnd` — when that input has focus, put the caret at the end and clear the selection (used after history recall in the console).
- `ImHyperlink(url, caption)` — underlined blue (.3,.5,.9) text with a hand cursor; click opens the system browser; tooltip shows the URL when a caption is given.
