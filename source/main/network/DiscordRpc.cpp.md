# source/main/network/DiscordRpc.cpp

> Publishes single- or multiplayer status with the terrain name.

**Needs** — [`DiscordRpc.h`](DiscordRpc.h.md) · [`AppContext.h`](../AppContext.h.md) · [Seam: Discord rich presence](../../../SYSTEM-REQUIREMENTS.md#seam-discord-rich-presence)
**Used by** — callers of [`DiscordRpc.h`](DiscordRpc.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`DiscordRpc.h`](DiscordRpc.h.md).

## State

Stateless.

## `Init`

Registers application id `492484203435393035` with ready/error handlers that only log.

## `UpdatePresence`

Connected → state "Playing online", details "On server: host:port  on terrain: <name>"; else "Playing singleplayer", "On terrain: <name>". Start timestamp = now (so the elapsed timer restarts on every update), large image key `ror_logo_t`, text "Rigs of Rods".
