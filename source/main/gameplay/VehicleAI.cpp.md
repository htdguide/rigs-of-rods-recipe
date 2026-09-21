# source/main/gameplay/VehicleAI.cpp

> Steering, speed control and simple collision avoidance toward the next waypoint, per vehicle kind and AI mode.

**Needs** — [`VehicleAI.h`](VehicleAI.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`system/Console.h`](../system/Console.h.md) · [`Engine.h`](Engine.h.md) · [`GameContext.h`](../GameContext.h.md) · [`physics/air/AeroEngine.h`](../physics/air/AeroEngine.h.md) · [`physics/water/ScrewProp.h`](../physics/water/ScrewProp.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`gui/panels/GUI_TopMenubar.h`](../gui/panels/GUI_TopMenubar.h.md)
**Used by** — callers of [`VehicleAI.h`](VehicleAI.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

A deliberately naive "carrot" follower: steer toward the target in the vehicle's frame, regulate speed by bang-bang throttle/brake, slow for sharp turns and obstacles. Behaviour depends on the AI panel's mode: 0 normal, 1 race, 2 drag race, 3 crash, 4 chase (target = the player).

## State

See [`VehicleAI.h`](VehicleAI.h.md).

## `update(dt)` — once per frame while active

```text
IF waiting: count down; RETURN
work in the horizontal plane (y = 0)
reach radius = 5 m (boats 50, aircraft 100)
turn angle = angle between (current − prev) and (next − current)          # used in normal mode
IF chase mode: target = player's actor or character position
ELSE IF any node within the reach radius of the target: update_waypoint(); RETURN
local = inverse(actor heading) · (target − position), normalised
yaw = local.x; IF local.z > 0 (target behind): yaw = ±1                     # full lock to turn around
trucks: steering = yaw; boats: rudders = −yaw
aircraft: until the last waypoint steer wheels and ailerons (yaw/2, zero if |roll| > 0.5); when |yaw| < 0.1 or at the end, level the wings
```

**Trucks** — start the engine and release the parking brake; with speed in km/h:

| | below max − 1 | above max + 1 | otherwise |
|---|---|---|---|
| nearly straight (|yaw| < 0.5) | throttle `acc_power − 0.1·turn_angle_rad`, no brake | throttle 0, brake 1/3 | coast |
| turning | throttle `acc_power/3` | throttle 0, brake 1/2 | coast |

Normal mode, when the previous waypoint's speed is "auto" (−1): if within *speed-in-km/h* metres of a turn, `max = (1 − t)·50 + t·5` with `t = 1.4·(angle° − 10)/170`, capped at 50 and at the panel speed; else the panel speed. Obstacles within 30° ahead: another driveable actor closer than the speed value → full brake; any node pairs closer than 5 m → parking brake and flash lights; the walking player closer than the speed value → brake, closer than 5 m → steer hard left and flash. Race/drag/crash modes only reset the speed to the panel value. Chase mode adds the player's speed to the max, stops within 10 m of actors ahead or 20 m of the player on foot.

**Aircraft** — release parking brake, start every engine at full throttle; climb with `elevator = 0.5 − |pitch|` and flaps 4× that (elevator −0.05 if pitch > 0.5) until reaching the panel altitude above the start height, then throttle 0.9 and *hold*: `elevator = −pitch`, flaps 1 when nose-down else 0; resume climbing below 80 % of the target.

**Boats** — speed in knots along the heading from the camera node; the same bang-bang table with screw-prop throttles (no brake).

## `updateWaypoint`

Console notice "Reached waypoint: <name>"; run its event (lights toggle, beacons toggle — horn and wait are declared but not implemented); apply its speed and power overrides; advance. After the last waypoint: index wraps to 0, last flag set; trucks and boats deactivate (trucks set the parking brake, boats reset props); aircraft keep flying. Trucks refresh prev/next waypoint positions.

## `getTranslation(offset, waypoint)`

Offset for spawning several AI vehicles in formation (panel's position scheme 0 = in line behind, 1 = side by side): for the first waypoint relative to the vehicle's heading, else relative to the direction between the panel's waypoint and the previous one.

**Notes** — speeds set with `setValueAtWaypoint(SPEED)` are keyed by waypoint *name*, powers by index; a name-less waypoint therefore cannot carry a speed. The panel stores −1 for "automatic" speed.
