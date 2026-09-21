# source/main/network/DiscordRpc.h

> Discord "now playing" status.

**Needs** — [Seam: Discord rich presence](../../../SYSTEM-REQUIREMENTS.md#seam-discord-rich-presence)
**Used by** — [`Application.cpp`](../Application.cpp.md) · [`main.cpp`](../main.cpp.md) · [`DiscordRpc.cpp`](DiscordRpc.cpp.md)
**Tier floor** — T2


## Purpose

Optional [Seam: Discord rich presence](../../../SYSTEM-REQUIREMENTS.md#seam-discord-rich-presence); every call is a no-op without it or when the `io_discord_rpc` setting is off. Implementation: [`DiscordRpc.cpp`](DiscordRpc.cpp.md).

## State

Stateless.

## `Init` / `UpdatePresence` / `Shutdown`

Called at startup, on terrain load / multiplayer changes, and at exit.
