# source/main/gameplay/Character.h

> The walking player avatar (local or remote): kinematic movement, animation state, vehicle coupling, network sync; plus its graphics companion.

**Needs** — [`ForwardDeclarations.h`](../ForwardDeclarations.h.md) · [`terrain/SurveyMapEntity.h`](../terrain/SurveyMapEntity.h.md)
**Used by** — [`audio/MumbleIntegration.cpp`](../audio/MumbleIntegration.cpp.md) · [`Character.cpp`](Character.cpp.md) · [`CharacterFactory.cpp`](CharacterFactory.cpp.md) · [`CharacterFactory.h`](CharacterFactory.h.md) · [`gfx/camera/CameraManager.cpp`](../gfx/camera/CameraManager.cpp.md) · [`gui/OverlayWrapper.cpp`](../gui/OverlayWrapper.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`system/ConsoleCmd.cpp`](../system/ConsoleCmd.cpp.md)
**Tier floor** — T2


## Purpose

When not driving, the player is a character: a kinematic point (no soft body) that walks, runs, swims and jumps, collides with the static world and stands on actor cabs. Remote players are characters driven by network packets. `GfxCharacter` is the render-side twin fed through a double buffer. Implementation: [`Character.cpp`](Character.cpp.md).

## State

```text
RECORD Character
  position, previous position, rotation (rad), h_speed = 2 m/s, v_speed, can_jump
  actor_coupling : Actor?          # the vehicle it sits in (then it follows the driver seat)
  remote, source_id, stream_id, net username, colour number, instance name ("Character#n")
  anim_name = "Idle_sway", anim_time, last sent anim time, driving animation length
  net timer, last update time

RECORD GfxCharacter
  scene node (mesh "character.mesh", scale 0.02, own material clone), character
  sim buffer (current, previous): position, rotation, username, remote, colour, coupling, anim name/time
  survey-map entity
```

## API

Getters, `setPosition` (also resets previous position), `setRotation`, `move(offset)`, `update(dt)`, `receiveStreamData`, `SetActorCoupling(enabled, actor)`, `SetupGfx`; graphics: `BufferSimulationData`, `UpdateCharacterInScene`.
