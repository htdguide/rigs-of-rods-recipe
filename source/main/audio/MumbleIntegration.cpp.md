# source/main/audio/MumbleIntegration.cpp

> Maps the MumbleLink shared memory and writes positional context each frame.

**Needs** — [`MumbleIntegration.h`](MumbleIntegration.h.md) · [`Application.h`](../Application.h.md) · [`gfx/camera/CameraManager.h`](../gfx/camera/CameraManager.h.md) · [`gameplay/Character.h`](../gameplay/Character.h.md) · [`GameContext.h`](../GameContext.h.md)
**Used by** — callers of [`MumbleIntegration.h`](MumbleIntegration.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`MumbleIntegration.h`](MumbleIntegration.h.md).

## State

See header.

## Mapping the link

Windows: open the existing named file mapping `MumbleLink` and map a view of it. POSIX: open shared-memory object `/MumbleLink.<uid>` read-write and map it shared. Any failure → no link. The block is created by Mumble, never by the game.

## `Update()`

```text
IF in simulation AND connected to a multiplayer server:
  avatar dir = (cos r, 0, sin r) of the character's rotation r
  camera dir/up = camera orientation × (−Z / +Y)
  write(camera pos, camera dir, camera up, character pos + (0, 1.8, 0), avatar dir, +Y)
ELSE:
  write(origin, +Z, +Y, origin, +Z, +Y)          # non-positional: everyone sounds centred
```

**Notes** — the "is there a player character" check is absent; the connected-simulation state implies one.

## `write` (updateMumble)

```text
IF no link: RETURN
IF link.uiVersion ≠ 2: set name "Rigs of Rods", description, uiVersion = 2
uiTick += 1                                     # Mumble treats a stalled tick as "game gone"
normalise all direction vectors
copy positions/vectors with Z negated           # engine is right-handed, Mumble left-handed
identity = multiplayer player name
context  = "<server host>:<port or 1337>|0"    # 0 = team; players hear each other only with equal context
context_len = length(context)
```

**Notes** — the context string is what makes Mumble group only players on the same server; keep its format if interoperating with existing clients. Unmapping is not done on shutdown (the destructor frees the pointer as if heap-allocated, which is a bug; a rebuild unmaps it).
