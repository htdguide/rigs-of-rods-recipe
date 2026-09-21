# source/main/gameplay/VehicleAI.h

> Waypoint-following AI driver for trucks, boats and aircraft (script-controlled; only built with the script engine).

**Needs** — [`Application.h`](../Application.h.md) · [`utils/memory/RefCountingObject.h`](../utils/memory/RefCountingObject.h.md)
**Used by** — [`GameContext.cpp`](../GameContext.cpp.md) · [`VehicleAI.cpp`](VehicleAI.cpp.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`scripting/ScriptEngine.cpp`](../scripting/ScriptEngine.cpp.md) · [`scripting/bindings/VehicleAiAngelscript.cpp`](../scripting/bindings/VehicleAiAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

Every actor gets one (when scripting is compiled in); it is inactive until a script or the AI panel fills waypoints and activates it. While active it overrides the actor's controls and the actor manager skips player-style truck features and sleeping for it. Implementation: [`VehicleAI.cpp`](VehicleAI.cpp.md). The method order mirrors the script binding.

## State

```text
ENUM event = HORN | LIGHTSTOGGLE | WAIT_SECONDS | BEACONSTOGGLE
ENUM value = SPEED | POWER
RECORD VehicleAI
  actor, enabled, waiting, wait_time
  waypoints : map<1..n, Vec3>; ids : map<name, index>; names : map<index, name>
  events : map<index, event>; speed : map<name, km/h>; power : map<index, 0..1>
  current/prev/next waypoint (positions), current index (starts 0), count
  max_speed = 50 km/h, acc_power = 0.8, init_y (height when activated), last_waypoint, hold (aircraft altitude hold)
```

## API

`setActive(bool)` (records starting height), `isActive`, `addWaypoint(name, pos)` (first one becomes the current target), `addWaypoints(dictionary)`, `addEvent(name, event)`, `setValueAtWaypoint(name, SPEED|POWER, value)`, `getTranslation(offset, waypoint)`, `update(dt, do_update)`.
