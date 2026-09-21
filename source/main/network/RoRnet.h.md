# source/main/network/RoRnet.h

> The RoRnet 2.45 multiplayer wire protocol: message codes, flag sets and packed record layouts shared with the server.

**Needs** — [`utils/BitFlags.h`](../utils/BitFlags.h.md)
**Used by** — [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`gui/panels/GUI_CollisionsDebug.cpp`](../gui/panels/GUI_CollisionsDebug.cpp.md) · [`gui/panels/GUI_GameAbout.cpp`](../gui/panels/GUI_GameAbout.cpp.md) · [`gui/panels/GUI_GameMainMenu.cpp`](../gui/panels/GUI_GameMainMenu.cpp.md) · [`gui/panels/GUI_MultiplayerClientList.h`](../gui/panels/GUI_MultiplayerClientList.h.md) · [`gui/panels/GUI_MultiplayerSelector.cpp`](../gui/panels/GUI_MultiplayerSelector.cpp.md) · [`gui/panels/GUI_TopMenubar.h`](../gui/panels/GUI_TopMenubar.h.md) · [`Network.h`](Network.h.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`utils/Utils.cpp`](../utils/Utils.cpp.md)
**Tier floor** — T2


## Purpose

Wire vocabulary shared byte-for-byte with the external server software ([Data and persistence](../../../SYSTEM-REQUIREMENTS.md#5-data-and-persistence)). Everything here is fixed by interop; nothing is a free choice.

## State

Stateless — constants and layouts.

## Constants

max peers 64 · max message 8192 bytes (header included) · LAN broadcast port 13000 · username 40 bytes · version string `"RoRnet_2.45"`.

## Encoding

All records are packed (no padding), little-endian, fixed-width integers and IEEE-754 32-bit floats; strings are fixed-size NUL-padded byte arrays (UTF-8). Flag sets are 32-bit masks where "bit n" means value `1 << (n−1)`.

## `MessageType`

Consecutive codes from 1025: HELLO, FULL, WRONG_PW, WRONG_VER, BANNED, WELCOME, VERSION, SERVER_SETTINGS, USER_INFO, MASTERINFO, NETQUALITY, GAME_CMD, USER_JOIN, USER_LEAVE, UTF8_CHAT, UTF8_PRIVCHAT, STREAM_REGISTER, STREAM_REGISTER_RESULT, STREAM_UNREGISTER, STREAM_DATA, STREAM_DATA_DISCARDABLE, NO_RANK (so HELLO=1025 … NO_RANK=1046). Legacy: WRONG_VER_LEGACY = 1003 (servers ≤ 2.38).

## Flag sets

- **UserAuth** — ADMIN bit 1, RANKED 2, MOD 3, BOT 4, BANNED 5 (NONE = 0).
- **Netmask** (vehicle state) — HORN 1, POLICEAUDIO 2, PARTICLE 3, PBRAKE 4, TC_ACTIVE 5, ALB_ACTIVE 6, ENGINE_CONT (ignition) 7, ENGINE_RUN 8, engine mode AUTOMATIC 9, SEMIAUTO 10, MANUAL 11, MANUAL_STICK 12, MANUAL_RANGES 13.
- **Lightmask** — CUSTOM1..10 bits 1–10, HEADLIGHT 11, HIGHBEAMS 12, FOGLIGHTS 13, SIDELIGHTS 14, BRAKES 15, REVERSE 16, BEACONS 17, BLINK_LEFT 18, BLINK_RIGHT 19, BLINK_WARN 20.
- **PeerOptions** (client-local, never sent) — MUTE_CHAT 1, MUTE_ACTORS 2, HIDE_ACTORS 3.
- **UiStreamsHealth** (client-local) — INVALID −1, MISMATCHES 0, ALL_OK 1, IDLE 2.

## Records

```text
Header (16)            command u32, source i32 (0 = server), streamid u32, size u32 (payload bytes)
StreamRegister (272)   type i32 (0 actor, 1 character, 3 chat), status i32, origin_sourceid i32, origin_streamid i32,
                       name[128], data[128]
ActorStreamRegister    same first 144 bytes; data[128] reinterpreted as
                       bufferSize i32, time i32, skin[60], sectionconfig[60]
StreamUnRegister (4)   streamid u32
UserInfo (359)         uniqueid u32, authstatus i32, slotnum i32, colournum i32, username[40], usertoken[40],
                       serverpassword[40], language[10], clientname[10], clientversion[25], clientGUID[40],
                       sessiontype[10], sessionoptions[128]
VehicleState (40)      time i32, engine_speed f32 (RPM), engine_force f32, engine_clutch f32, engine_gear i32,
                       hydrodirstate f32 (steering), brake f32, wheelspeed f32, flagmask u32, lightmask u32
ServerInfo (4373)      protocolversion[20], terrain[128], servername[128], has_password u8, info[4096]
LegacyServerInfo (20)  protocolversion[20]
```

**Notes** — `VehicleState` is the header of every actor stream-data packet; the node positions that follow it are defined by [`physics/Actor.cpp`](../physics/Actor.cpp.md). The character and chat stream payloads are defined in [`Network.h`](Network.h.md) and [`gameplay/ChatSystem.cpp`](../gameplay/ChatSystem.cpp.md).
