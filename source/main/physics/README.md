# source/main/physics

> The soft-body simulation: actors made of nodes and beams, stepped at 2 kHz, colliding with the world and each other, flying, floating and deforming.

This is the heart of the program and the reason for its [tier](../../../SYSTEM-REQUIREMENTS.md#2-tier). Everything else either feeds it (truck files, input) or reads from it (graphics, sound, network).

## The model in one paragraph

An **actor** is a set of point masses (*nodes*) joined by damped springs (*beams*). Each physics step (0.5 ms) computes beam forces with plastic deformation and breaking, adds shocks, hydraulics, commands, wheels and drivetrain, aerodynamics and buoyancy, resolves contacts against terrain, static objects and other actors, and integrates node velocities and positions (symplectic Euler). A frame runs as many whole steps as its elapsed time allows (capped at 1/20 s); the remainder carries to the next frame. Actors at rest go to sleep after 10 s and are woken by approaching neighbours.

## Structure

- **Data** — [`SimConstants.h`](SimConstants.h.md), [`SimData.h`](SimData.h.md) (node, beam, shock, wheel, hook, tie, rope, command… records), [`ApproxMath.h`](ApproxMath.h.md).
- **The actor** — [`Actor.h`](Actor.h.md) / [`Actor.cpp`](Actor.cpp.md) (lifecycle, network, resets, linking, shocks and triggers, lights), [`ActorForcesEuler.cpp`](ActorForcesEuler.cpp.md) (the per-step force loop), [`ActorSlideNode.cpp`](ActorSlideNode.cpp.md), [`ActorExport.cpp`](ActorExport.cpp.md).
- **Building actors** — [`ActorSpawner.h`](ActorSpawner.h.md), [`ActorSpawnerFlow.cpp`](ActorSpawnerFlow.cpp.md) (order), [`ActorSpawner.cpp`](ActorSpawner.cpp.md) (meaning of each truck-file element).
- **Running all actors** — [`ActorManager.h`](ActorManager.h.md) / [`ActorManager.cpp`](ActorManager.cpp.md) (frame → steps, threading, sleep, multiplayer streams, free forces), [`Savegame.cpp`](Savegame.cpp.md).
- **Mechanisms** — [`SlideNode`](SlideNode.h.md), [`Differentials`](Differentials.h.md), [`CmdKeyInertia`](CmdKeyInertia.h.md).
- **Sub-chapters** — [`collision/`](collision/README.md), [`air/`](air/README.md), [`water/`](water/README.md), [`flex/`](flex/README.md).

## Reading order

1. [`SimConstants.h`](SimConstants.h.md), [`ApproxMath.h`](ApproxMath.h.md), [`SimData.h`](SimData.h.md), [`SimData.cpp`](SimData.cpp.md)
2. [`CmdKeyInertia`](CmdKeyInertia.h.md), [`Differentials`](Differentials.h.md), [`SlideNode`](SlideNode.h.md)
3. [`collision/`](collision/README.md), [`air/`](air/README.md), [`water/`](water/README.md), [`flex/`](flex/README.md)
4. [`Actor.h`](Actor.h.md) → [`ActorForcesEuler.cpp`](ActorForcesEuler.cpp.md) → [`Actor.cpp`](Actor.cpp.md) → [`ActorSlideNode.cpp`](ActorSlideNode.cpp.md)
5. [`ActorSpawner.h`](ActorSpawner.h.md) → [`ActorSpawnerFlow.cpp`](ActorSpawnerFlow.cpp.md) → [`ActorSpawner.cpp`](ActorSpawner.cpp.md)
6. [`ActorManager.h`](ActorManager.h.md) → [`ActorManager.cpp`](ActorManager.cpp.md) → [`Savegame.cpp`](Savegame.cpp.md) → [`ActorExport.cpp`](ActorExport.cpp.md)

## Cycles

The actor refers to gameplay (engine, autopilot, AI), graphics (graphics actor, flexbodies) and audio, which all refer back to the actor. The recipe breaks the cycle here: read the actor with those parts as named collaborators whose own chapters come later; their contracts are all the actor relies on.
