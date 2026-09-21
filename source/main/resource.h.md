# source/main/resource.h

> Windows resource id of the application icon.

**Needs** — nothing
**Used by** — the Windows resource script (build-time only)
**Tier floor** — T4

## Purpose

Declares icon id `101`, which [`AppContext::SetUpRendering`](AppContext.cpp.md#setuprendering) loads to set the window icon on Windows. Platform packaging detail; a rebuild sets its window icon however its windowing layer prefers.

## State

Stateless.
