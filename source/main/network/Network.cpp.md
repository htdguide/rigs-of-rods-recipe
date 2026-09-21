# source/main/network/Network.cpp

> Handshake, framing, and the send/receive threads of the multiplayer client.

**Needs** — [`Network.h`](Network.h.md) · [`Application.h`](../Application.h.md) · [`gameplay/ChatSystem.h`](../gameplay/ChatSystem.h.md) · [`system/Console.h`](../system/Console.h.md) · [`utils/ErrorUtils.h`](../utils/ErrorUtils.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`gui/panels/GUI_TopMenubar.h`](../gui/panels/GUI_TopMenubar.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`RoRVersion.h`](../../../source/version_info/RoRVersion.h.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [Seam: TCP sockets](../../../SYSTEM-REQUIREMENTS.md#seam-tcp-sockets)
**Used by** — callers of [`Network.h`](Network.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`Network.h`](Network.h.md).

## State

See header.

## Framing

A message is a 16-byte header followed by `size` payload bytes; header + payload < 8192. Receiving reads exactly the header, rejects a size above the buffer, then reads exactly the payload. Any short read is a connection error.

## Connect thread

```text
socket timeout 10 s; TCP connect(host, port)                  fail → CONNECT_FAILURE
send HELLO(payload = "RoRnet_2.45")
receive:
  WRONG_VER_LEGACY → fail, showing the legacy server's version (or "≤2.38")
  WRONG_VER → fail; not HELLO → fail
  HELLO: payload is ServerInfo; its protocolversion must start with our version, else fail
timeout off (blocking)
send USER_INFO: username (≤40 bytes), serverpassword = SHA-1 hex of password, usertoken = SHA-1 hex of token,
                clientversion = game version, clientname "RoR", language "ll_CC", sessiontype "normal"
receive: FULL | BANNED (do not close the socket) | WRONG_PW | WRONG_VER | NO_RANK (also open MP settings) → fail
         anything but WELCOME → fail
uid = header.source; local user = payload
start send and receive threads; CONNECT_SUCCESS
```

Progress is reported as messages at each step; failure logs host/port and closes the socket (1 s timeout).

## Send thread

Waits on the condition variable for a queued packet or shutdown; sends each whole. A send error is logged and ignored.

## `AddPacket(streamid, type, len, bytes)`

Payload over 8192 − 16 → dropped with a log line. Builds header (source = own uid). For STREAM_DATA_DISCARDABLE: if more than 20 packets are queued, drop it; if a queued packet has an identical header (same stream, type and size), overwrite it in place — only the newest state of a stream is worth sending. Otherwise append. Notify the sender.

## `AddLocalStream(register, size)`

Stamps origin = (own uid, next stream id), status 0, sends STREAM_REGISTER on that stream id, and increments the id.

## Receive thread

```text
LOOP until shutdown:
  error → shutdown, RECV_ERROR message, stop
  STREAM_REGISTER from self → ignore; other stream (un)registrations and chat → queued for the game thread
  NETQUALITY from source −1 with a 4-byte payload → store quality
  USER_LEAVE:
    about self → shutdown; kicked unless the text contains "disconnected on request"; post KICK or USER_DISCONNECT
    about a peer → console "left the game", move peer to disconnected list, remove its peer-options slot
  USER_INFO / USER_JOIN:
    about self → replace local user, auth level, name
    new peer → console "(Auth) joined the game", append peer and a zero peer-options mask
  GAME_CMD → hand the text to the script engine
  anything else → queue for the game thread
  (after every membership change: post "refresh client list")
```

## `Disconnect`

Clean if not already shut down. Set shutdown, wake and join the sender; with a 1 s timeout send USER_LEAVE (clean case only); join the receiver; close gracefully (clean) or just drop the descriptor. Clear users, queues and quality, clear networked console lines, state DISABLED.

## Chat

Broadcast: UTF8_CHAT with the text. Whisper: UTF8_PRIVCHAT with payload = target uid (u32) + text.

**Notes** — the original builds the whisper payload but then sends the bare text with the payload's length, so whispers go out malformed; a rebuild sends the payload. `hash` in debug packet logging is diagnostic only.
