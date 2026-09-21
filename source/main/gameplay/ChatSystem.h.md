# source/main/gameplay/ChatSystem.h

> Multiplayer chat stream: registration and receiving.

**Needs** — [`Application.h`](../Application.h.md) · [`network/Network.h`](../network/Network.h.md)
**Used by** — [`ChatSystem.cpp`](ChatSystem.cpp.md) · [`gui/panels/GUI_GameChatBox.cpp`](../gui/panels/GUI_GameChatBox.cpp.md) · [`main.cpp`](../main.cpp.md) · [`network/Network.cpp`](../network/Network.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md)
**Tier floor** — T2


## Purpose

Chat rides on its own network stream. Sending chat text is done by the network layer; this module registers the stream and turns received chat packets into console messages. Implementation: [`ChatSystem.cpp`](ChatSystem.cpp.md).

## State

Stateless.

## API

`SendStreamSetup()`, `HandleStreamData(packets)`.
