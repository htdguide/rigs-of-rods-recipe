# source/main/utils/ErrorUtils.cpp

> Implements the OS-level message boxes.

**Needs** — [`ErrorUtils.h`](ErrorUtils.h.md) · [`Utils.h`](Utils.h.md) · [`Language.h`](Language.h.md)
**Used by** — callers of [`ErrorUtils.h`](ErrorUtils.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Platform-specific display of fatal and informational messages.

## State

Stateless.

## `ShowError`

**Contract** — logs `"[RoR|ErrorPopupDialog] <title>: <message>"`, then shows a box titled "FATAL ERROR" (translated where translation is available) whose body is "An internal error occured in Rigs of Rods. Technical details below:" followed by the message. The caller's `title` goes to the log only.

## `ShowInfo`

**Contract** — shows the message with the caller's title, no logging.

## `ShowMsgBox`

**Contract** — Windows: a topmost modal box with an error or info icon, text converted UTF-8 → UTF-16. Linux/macOS: prints `title: message` to standard output. Must work before any other subsystem is initialised.
