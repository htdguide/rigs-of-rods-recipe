# source/main/gfx/camera/CameraManager.cpp

> Behaviour switching and the math of each camera mode.

**Needs** — [`CameraManager.h`](CameraManager.h.md) · [`AppContext.h`](../../AppContext.h.md) · [`physics/ApproxMath.h`](../../physics/ApproxMath.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`physics/ActorManager.h`](../../physics/ActorManager.h.md) · [`gameplay/Character.h`](../../gameplay/Character.h.md) · [`physics/collision/Collisions.h`](../../physics/collision/Collisions.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`GfxScene.h`](../GfxScene.h.md) · [`utils/InputEngine.h`](../../utils/InputEngine.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`gui/OverlayWrapper.h`](../../gui/OverlayWrapper.h.md) · [`gameplay/Replay.h`](../../gameplay/Replay.h.md) · [`terrain/Terrain.h`](../../terrain/Terrain.h.md) · [`gui/GUIManager.h`](../../gui/GUIManager.h.md) · [`PerVehicleCameraContext.h`](PerVehicleCameraContext.h.md) · [`GfxWater.h`](../GfxWater.h.md)
**Used by** — callers of [`CameraManager.h`](CameraManager.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`CameraManager.h`](CameraManager.h.md).

## State

See header.

## Per frame — `UpdateInputEvents(dt)` (only while simulating or editing)

```text
refresh context (player actor, simulation speed, scales: rotation 50·dt degrees, translation 100·dt)
IF the actor forces a cinecam: switch to CINECAM and select it
ELSE
  camera-change key (in cyclic modes): if the current mode agrees to leave, go to the next mode (mod END)
     CHARACTER: third → first person stays in the mode; first → leave
     CINECAM: advance to the next cinecam; after the last one, leave
  free-fixed key toggles FIXED, free key toggles FREE (see toggling)
update the current behaviour (none → CHARACTER)
FOV keys (not static mode): ±1° on the internal (cinecam) or external setting, limited to 10..160 (console notice); reset key restores the default
```

Activation rules: CHARACTER only without a vehicle; VEHICLE needs a vehicle; SPLINE needs camera rails; CINECAM needs cinecams (else skip to the next). Activation stores the mode in the vehicle's context, handles the FOV (internal for cinecam, restored on leaving), switches the vehicle's interior mode (hide exterior, show dashboard overlay for aircraft), and notifies the actor for sound/visibility. Changing vehicles restores that vehicle's remembered mode unless the camera is FREE, FIXED or STATIC; leaving a vehicle goes to CHARACTER under the same exception.

**Toggling** FREE/FIXED: entering remembers the previous mode (or, if already toggled, the other toggled mode in a one-slot history); pressing the same key again returns.

## Orbit camera (character, vehicle, spline, and cinecam look-around)

```text
look-back key flips rot_x between 0 and 180°; arrow keys rotate (optionally inverted), pitch clamped −80°..88°
zoom keys ±translation scale (×10 fast); reset key resets the mode; Right-Shift+Space toggles movement limits
limits: distance ≥ min (and ≤ max) when limited; ≥ 0
desired = look_at + dist/2 · (sin(dir+rx)·cos(pitch+ry), sin(pitch+ry), cos(dir+rx)·cos(pitch+ry))
limited: keep ≥ 1 m above terrain
smoothing k = ratio: position = desired/(k+1) + k/(k+1)·(previous position + look-at displacement);
  look-at smoothed the same way
event box forcing a camera position overrides (one frame); replaying with movement → no smoothing
look at the smoothed point
```

Mouse (right button): rotate 0.13°/pixel, wheel zoom (fine with Alt). Right+middle: reset.

**Character** — direction behind the character; look-at at 1.1 m (third person) or 1.82 m (first person); first person: mouse turns the character and pitches (−0.9·90°..0.65·90°), distance 0.1, no smoothing.

**Vehicle** — target direction behind the vehicle's heading; pitching mode (`gfx_extcam_mode`) follows vehicle pitch; smoothing ratio `1/(4·dt)`; min distance `min(2·min camera radius, 33)`; reset: pitch 0.35, distance 1.5·min + 2. Shift + middle click on a custom orbit node re-derives angles and distance from the current camera so the view does not jump.

**Spline (camera rails)** — spline through the vehicle's `camerarail` nodes, extended with linked actors' rails whose ends are within 5 m of either end (appended, prepended or reversed as needed); closed when ends within 1 m; length = half the sum of segment lengths; points follow the nodes each frame; look-at = spline(position); Ctrl + right-drag slides along it (wrapping if closed); Left-Shift+Space toggles auto tracking (turn toward the vehicle centre). Optional debug line.

**Cinecam** — camera at the cinecam node; basis from the camera direction node and roll node (roll flipped if the definition was corrected at spawn); orientation `rot_x about up · (180° + rot_y) about roll · basis`; reset pitch −15°.

**Free** — keys move (sidestep, forward/back, up/down) and turn (yaw in world, pitch local), scaled by Shift ×3/×5, Ctrl ×6/×10, Alt ×0.2; mouse yaw/pitch 0.13°/pixel.

**Fixed** — stays put; optionally tracks the vehicle or character (`gfx_fixed_cam_tracking`).

**Static** ("TV camera"):

```text
target = vehicle (or character) position; velocity scaled by simulation speed; radius = min camera radius (3 for aircraft)
re-place the camera when forced, or every second if: far (> 8·r) and the target moves toward it, or near (< 2·r) and moving away,
  or farther than max(25, 1.15·speed)·r, or terrain/objects block the line of sight to the target or its predicted position
candidates (up to 10): slow → random point on a circle 2.5·r; fast → ahead along the velocity with random lateral offset;
  lifted above water and terrain plus max(2.89·√r, camera height) (the last 3 with random height);
  keep those with a clear view; stop at a good height match or 3 candidates; choose the best height match
FOV = atan2(20, distance^exponent) with the exponent smoothed (right-drag + wheel adjusts 0.8..1.5)
```

Line-of-sight test: sample the segment every half metre against terrain height, then ray-test collision triangles; sampled for several end points between the target and its prediction.

**Notes** — the "rotation" scale uses the translation-speed constant and vice versa (50 vs 100); the camera reads the live actor (not the sim buffer) — acceptable because it runs on the main thread after synchronisation.
