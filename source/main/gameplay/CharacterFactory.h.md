# source/main/gameplay/CharacterFactory.h

> Owns the local character and remote players’ characters.

**Needs** — [`Application.h`](../Application.h.md) · [`Character.h`](Character.h.md) · [`network/Network.h`](../network/Network.h.md)
**Used by** — [`GameContext.h`](../GameContext.h.md) · [`CharacterFactory.cpp`](CharacterFactory.cpp.md)
**Tier floor** — T2


## Purpose

One per game context. Implementation: [`CharacterFactory.cpp`](CharacterFactory.cpp.md).

## State

```text
RECORD CharacterFactory = { local : Character?; remotes : list<Character> }
```

## API

`CreateLocalCharacter`, `GetLocalCharacter`, `DeleteAllCharacters`, `UndoRemoteActorCoupling(actor)`, `Update(dt)`, `handleStreamData(packets)`.
