# source/main/gameplay

> Everything that makes the simulation a game: engines and gearboxes, driving aids, autopilot and AI, the walking player, repair, replay, races, chat, mouse interaction, and the script event catalogue.

These modules sit *on* the physics: they read actor state and write actor controls (throttle, brake, steering, lights), but never compute node forces themselves — except the engine's clutch torque, which the wheel step consumes.

## Reading order

1. [`ScriptEvents.h`](ScriptEvents.h.md) — vocabulary used everywhere
2. [`TorqueCurve`](TorqueCurve.h.md) → [`Engine`](Engine.h.md) → [`CruiseControl.cpp`](CruiseControl.cpp.md), [`TyrePressure`](TyrePressure.h.md)
3. [`AutoPilot`](AutoPilot.h.md), [`VehicleAI`](VehicleAI.h.md)
4. [`Landusemap`](Landusemap.h.md)
5. [`Character`](Character.h.md) → [`CharacterFactory`](CharacterFactory.h.md)
6. [`RepairMode`](RepairMode.h.md), [`Replay`](Replay.h.md), [`SceneMouse`](SceneMouse.h.md)
7. [`RaceSystem`](RaceSystem.h.md), [`ChatSystem`](ChatSystem.h.md)
8. [`RandomTreeLoader.h`](RandomTreeLoader.h.md) — unused
