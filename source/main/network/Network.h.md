# source/main/network/Network.h

> The multiplayer client: connection handshake, a send thread and a receive thread over one TCP socket, and the thread-safe peer list.

**Needs** — [`Application.h`](../Application.h.md) · [`RoRnet.h`](RoRnet.h.md)
**Used by** — [`Application.cpp`](../Application.cpp.md) · [`gameplay/Character.cpp`](../gameplay/Character.cpp.md) · [`gameplay/CharacterFactory.h`](../gameplay/CharacterFactory.h.md) · [`gameplay/ChatSystem.h`](../gameplay/ChatSystem.h.md) · [`gui/panels/GUI_ConsoleView.cpp`](../gui/panels/GUI_ConsoleView.cpp.md) · [`gui/panels/GUI_MultiplayerClientList.cpp`](../gui/panels/GUI_MultiplayerClientList.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`Network.cpp`](Network.cpp.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`system/ConsoleCmd.cpp`](../system/ConsoleCmd.cpp.md)
**Tier floor** — T2


## Purpose

Owns the connection to a RoRnet server. The game thread queues outgoing packets and drains incoming stream data once per frame; everything else (join/leave, chat notices, quality, script commands) is handled on the receive thread and reported to the game as messages. Compiled only with socket support. Implementation: [`Network.cpp`](Network.cpp.md).

## State

```text
RECORD Network
  socket
  server settings : ServerInfo; local user : UserInfo; uid : int; auth level
  users : list<UserInfo>, users_peeropts : list<mask>     # invariant: same length, same order (parallel)
  disconnected users : list<UserInfo>                      # kept so late packets/chat can still be attributed
  shadow copies of player name, host, port, password, token  # threads never read settings directly
  send queue : deque<SendPacket> + condition variable; recv queue : list<RecvPacket>
  shutdown : atomic bool; net quality : atomic int
  next local stream id : int = 10
  locks: users, local userdata, send queue, recv queue
```

## Local stream payloads (packed)

```text
character command i32: INVALID 0, POSITION 1, ATTACH 2, DETACH 3
CharacterMsgGeneric   command
CharacterMsgPos       command, x, y, z f32, rotation f32, anim_time f32, anim_name[10]
CharacterMsgAttach    command, source_id i32, stream_id i32, position i32   (seat on a remote actor)
SendPacket            buffer[8192] (header + payload), size
RecvPacket            header, buffer[8192]
```

## API

- `StartConnecting` → launches the connect thread; state CONNECTING. `StopConnecting` → state DISABLED, joins it.
- `Disconnect`, `AddPacket(streamid, type, len, bytes)`, `AddLocalStream(register, size)`, `GetIncomingStreamData() → packets` (takes and clears the queue).
- Queries (each under the users lock, returning copies): `GetUID`, `GetNetQuality`, `GetTerrainName`, `GetUserColor`, `GetUsername`, `GetLocalUserData`, `GetUserInfos`, `GetAllUsersPeerOpts`, `GetUserInfo(uid)`, `GetUserPeerOpts(uid)`, `GetDisconnectedUserInfo(uid)`, `GetAnyUserInfo(uid)` (remote, then local), `FindUserInfo(name)`.
- `GetPlayerColor(n)` — fixed palette of 25 classic colours; out of range → white.
- `AddPeerOptions` / `RemovePeerOptions` (set/clear mute/hide bits for one peer).
- `BroadcastChatMsg`, `WhisperChatMsg`, `UserAuthToStringShort/Long` (highest of admin > mod > bot > ranked > banned > guest).
