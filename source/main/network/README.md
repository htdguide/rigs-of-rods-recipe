# source/main/network

> Everything that crosses the machine boundary: the RoRnet multiplayer client, HTTP fetches, Discord presence and OutGauge telemetry.

[`RoRnet.h`](RoRnet.h.md) is the wire contract with the external server and must be matched exactly. [`Network`](Network.h.md) runs it: a connect thread does the handshake, then a send thread and a receive thread share one TCP socket, while the game thread only enqueues packets and drains received stream data once per frame. What the streams *carry* is defined by their owners: actor state in [`physics/Actor.cpp`](../physics/Actor.cpp.md), characters in [`gameplay/Character.cpp`](../gameplay/Character.cpp.md), chat in [`gameplay/ChatSystem.cpp`](../gameplay/ChatSystem.cpp.md).

The other three files are independent leaf integrations.

## Reading order

1. [`RoRnet.h`](RoRnet.h.md)
2. [`Network`](Network.h.md) ([impl](Network.cpp.md))
3. [`CurlHelpers`](CurlHelpers.h.md) ([impl](CurlHelpers.cpp.md))
4. [`OutGauge`](OutGauge.h.md) ([impl](OutGauge.cpp.md))
5. [`DiscordRpc`](DiscordRpc.h.md) ([impl](DiscordRpc.cpp.md))
