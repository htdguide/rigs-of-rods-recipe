# source/main

> The game itself. This directory holds the hub every chapter depends on, the chapters as subdirectories, and — read last — the game-state conductor and the program entry point.

**The hub.** [`Application.h`](Application.h.md) declares the vocabulary shared by everything: application/simulation/multiplayer states, every game message type, every setting, and accessors for the process-wide subsystems (created and destroyed in [`Application.cpp`](Application.cpp.md)). [`ForwardDeclarations.h`](ForwardDeclarations.h.md) names the id types and handles. [`AppContext`](AppContext.h.md) owns the window, renderer root, paths, logging, input setup and force feedback. [`pch.h`](pch.h.md), [`resource.h`](resource.h.md) and [`plugins.cfg.in`](plugins.cfg.in.md) are build plumbing.

**The conductor** (read after all chapters). [`GameContext`](GameContext.h.md) owns the message queue and the terrain/actor/character relationships; [`main.cpp`](main.cpp.md) fixes startup order and the per-frame order of message processing, input, scripts, audio, the physics hand-off and rendering.

## Reading order

1. [`ForwardDeclarations.h`](ForwardDeclarations.h.md)
2. [`Application.h`](Application.h.md) ([impl](Application.cpp.md))
3. [`AppContext`](AppContext.h.md) ([impl](AppContext.cpp.md)); [`pch.h`](pch.h.md), [`resource.h`](resource.h.md), [`plugins.cfg.in`](plugins.cfg.in.md)
4. Chapters: [`threadpool`](threadpool/README.md) · [`utils`](utils/README.md) · [`system`](system/README.md) · [`resources`](resources/README.md) · [`physics`](physics/README.md) · [`gameplay`](gameplay/README.md) · [`terrain`](terrain/README.md) · [`gfx`](gfx/README.md) · [`audio`](audio/README.md) · [`network`](network/README.md) · [`scripting`](scripting/README.md) · [`gui`](gui/README.md)
5. [`GameContext`](GameContext.h.md) ([impl](GameContext.cpp.md))
6. [`main.cpp`](main.cpp.md)

## Cycle

Every chapter includes `Application.h`, and `Application.h` names types from every chapter. The recipe breaks this at the hub: its twin is pure vocabulary and can be read first; the concrete subsystems it points to are read in their chapters.
