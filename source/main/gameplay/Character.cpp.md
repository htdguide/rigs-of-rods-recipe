# source/main/gameplay/Character.cpp

> Character movement and collision, animation selection, network messages, and scene update.

**Needs** — [`Character.h`](Character.h.md) · [`Application.h`](../Application.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`gfx/camera/CameraManager.h`](../gfx/camera/CameraManager.h.md) · [`physics/collision/Collisions.h`](../physics/collision/Collisions.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`utils/InputEngine.h`](../utils/InputEngine.h.md) · [`gfx/MovableText.h`](../gfx/MovableText.h.md) · [`network/Network.h`](../network/Network.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [`gfx/GfxWater.h`](../gfx/GfxWater.h.md)
**Used by** — callers of [`Character.h`](Character.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Movement is a hand-tuned kinematic controller, not physics — the character never pushes actors.

## State

See [`Character.h`](Character.h.md).

## `update(dt)` — local character on foot (not paused; skipped in free-camera mode)

```text
p = position; p.y += v·dt; v −= 9.8·dt                                     # gravity
collision_correct(p copy) with script callbacks                            # event boxes fire here
depth = height of the first collision-free point found scanning down from p + 0.3 in 1 mm steps (0 if none)
IF depth > 0: can_jump; v = max(0, v); p.y += min(depth, 2·dt)             # step up small ledges smoothly
stand on actors: for actors whose bounding box contains p, the highest collision-cab triangle hit by an upward ray
  within 1.8 m: can_jump; v = max(0, v); p.y += min(depth, 0.05)
obstacle sweep: if moved, test 100 points from previous + 0.25 m up to p; at the first collision stop one step earlier, +2.5 cm up
terrain: p.y = max(p.y, height), landing resets v and allows jumping
water: never deeper than 1.8 m below the wave surface; swimming when water depth > 1.8 m and p.y + 0.1 ≤ surface
input (jump: v = 2 m/s when allowed)
  turn right/left: rotation ± 2·dt·axis (×0.1 with Alt); "Turn" animation when standing
  sidestep: 0.5·h_speed·(run ? 3·run : axis) perpendicular; "Side_step" animation when standing
  forward (forward + look-up axes, ≤1): speed 1.5·h_speed·(run ? 3·run : axis); animation Swim_loop / Run / Walk with time dt·axis·h_speed
  backward: h_speed·axis backwards; Spot_swim / Walk played backwards
  idle: Spot_swim (2×) or Idle_sway
position = p
```

Heading convention: forward = (cos r, 0, sin r).

**Coupled to a vehicle** — animation "Driving" with its time set from the smoothed steering display: `((−steer + 1)/2)·length`, kept 0.01 away from the ends.

`SetAnimState(name, t)` restarts the animation at t when the name changes, else advances it by t.

## Network (multiplayer)

- Local character registers stream type 1, status 1, name "default", `data[0] = 2`, and remembers the assigned (source, stream).
- Every frame (at most every 100 ms, only when not in a vehicle) sends `POSITION`: x, y, z, rotation, animation name (fixed-length field) and the animation time advanced since the last send, as discardable stream data.
- `SetActorCoupling` sends `ATTACH {actor source, actor stream}` or `DETACH` (reliable stream data).
- Receiving (matching source and stream, reliable STREAM_DATA only): POSITION → set position, rotation, animation (if the name is terminated); DETACH → uncouple (error log if not coupled); ATTACH → couple to the remote actor with those network ids (error log if unknown); anything else → error log.

## Graphics

- `SetupGfx` — entity from `character.mesh` with infinite bounds (the animated mesh must never be culled), scale 0.02, hidden until updated, a private clone of `tracks/character` (for player colour); driving animation length recorded.
- `UpdateCharacterInScene` — entering a vehicle hides shadows and shows the character only if the vehicle has a driver-seat prop (and it is not a hidden network actor); in a vehicle it sits at the seat transform offset −0.6 m down; on foot it is placed at position with yaw −rotation. When the animation name changes enable only that animation and advance it; otherwise set its time. In multiplayer: tint by player colour and draw a name label above the head (unless hidden by the net-label options) at 1.9 m + camera distance/100.
