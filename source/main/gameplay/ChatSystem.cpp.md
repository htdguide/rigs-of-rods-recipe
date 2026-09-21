# source/main/gameplay/ChatSystem.cpp

> Chat stream setup and receiving (mute, whisper tagging).

**Needs** — [`ChatSystem.h`](ChatSystem.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`system/Console.h`](../system/Console.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`utils/Utils.h`](../utils/Utils.h.md)
**Used by** — callers of [`ChatSystem.h`](ChatSystem.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`ChatSystem.h`](ChatSystem.h.md).

## State

Stateless.

## `SendStreamSetup`

Registers a local stream with type 3, status 1, name "chat" (rest zeroed).

## `HandleStreamData(packets)`

For each packet of type UTF8_CHAT or UTF8_PRIVCHAT: unless the sender is the local user or the server (id −1), drop it when the sender is unknown or muted (peer option MUTE_CHAT); sanitise the UTF-8; prefix private messages with " [whispered] " (localised); post to the console as a network chat message from that sender.
