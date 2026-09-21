# source/main/gameplay/CharacterFactory.cpp

> Creating characters and routing multiplayer character streams.

**Needs** — [`CharacterFactory.h`](CharacterFactory.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`Application.h`](../Application.h.md) · [`Character.h`](Character.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`utils/Utils.h`](../utils/Utils.h.md)
**Used by** — callers of [`CharacterFactory.h`](CharacterFactory.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`CharacterFactory.h`](CharacterFactory.h.md).

## State

See header.

## Behaviour

- `CreateLocalCharacter` — in multiplayer uses the local user's colour and name, else colour −1 and no name; registers its graphics.
- `Update(dt)` — updates local then remote characters.
- `UndoRemoteActorCoupling(actor)` — uncouples every remote character sitting in that actor (called when the actor is deleted).
- `handleStreamData(packets)` — STREAM_REGISTER with type 1 → create a remote character for (source, stream) with that user's name and colour; USER_LEAVE → remove the *first* remote character from that source; anything else → offered to every remote character.
- `DeleteAllCharacters` — drops all.
