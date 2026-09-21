# Rigs of Rods — recipe

> Recipe of `rigs-of-rods` at `2cc94b1`, 2026-06-22 — branch `master` of the `htdguide/rigs-of-rods` fork (commit `2cc94b1856a9c50bf86af1513486ba8c70d2cb59`).

This repository is Rigs of Rods with the code taken out and the reasoning left in. Every C++ source file under `source/` has a markdown **twin** at the same path plus `.md`. The twin states what the file decides, owns and computes, in language-neutral pseudocode and prose, so the game can be rebuilt in another language. Nothing here is compiled.

## What Rigs of Rods is

A soft-body vehicle sandbox. Every vehicle is a cloud of point masses (nodes) joined by springs (beams), integrated at 2 kHz, so trucks, planes, boats and trains bend, break and interact physically. Around that core there is terrain streaming, an embedded script engine, a mod ecosystem of tens of thousands of community vehicles and maps, and a client for dedicated multiplayer servers.

## How to read it

1. [`SYSTEM-REQUIREMENTS.md`](SYSTEM-REQUIREMENTS.md) comes first. It covers the tier and its floor constraint, every **seam** (renderer, GUI toolkits, audio, script engine, sockets, HTTP…) with what a substitute must provide, platform assumptions, the file and wire formats a rebuild must keep compatible, and what counts as done.
2. [`GLOSSARY.md`](GLOSSARY.md) defines node, beam, actor, sim buffer, cache entry, script unit, stream and the other domain terms.
3. Then read the chapters in build order (below). Each directory's `README.md` opens its chapter and gives its reading order. Each twin has the same sections: a title, a one-line summary, **Needs** / **Used by** / **Tier floor**, then Purpose, State and one section per unit.

**Tier: T2 (managed native).** The floor constraint is the 2 kHz soft-body loop over about 10⁴ beams per vehicle, which must not allocate in steady state and must hand a consistent snapshot to the renderer every frame. See [SYSTEM-REQUIREMENTS §2](SYSTEM-REQUIREMENTS.md#2-tier).

## Tree

Twin counts per directory, READMEs excluded; 342 twins in total.

```
source/
├── main/  (11)
│   ├── audio/  (8)
│   ├── gameplay/  (29)
│   ├── gfx/  (33)
│   │   ├── camera/  (3)
│   │   └── particle/  (9)
│   ├── gui/  (10)
│   │   └── panels/  (50)
│   ├── network/  (9)
│   ├── physics/  (21)
│   │   ├── air/  (9)
│   │   ├── collision/  (8)
│   │   ├── flex/  (14)
│   │   └── water/  (6)
│   ├── resources/  (4)
│   │   ├── addonpart_fileformat/  (2)
│   │   ├── odef_fileformat/  (2)
│   │   ├── otc_fileformat/  (2)
│   │   ├── rig_def_fileformat/  (14)
│   │   ├── skin_fileformat/  (2)
│   │   ├── terrn2_fileformat/  (2)
│   │   ├── tobj_fileformat/  (2)
│   │   └── tuneup_fileformat/  (2)
│   ├── scripting/  (9)
│   │   └── bindings/  (23)
│   ├── system/  (8)
│   ├── terrain/  (15)
│   ├── threadpool/  (1)
│   └── utils/  (27)
│       ├── bbcode/  (2)
│       └── memory/  (2)
├── microbenchmarks/  (1)
└── version_info/  (2)
```

## Build order

1. [`source/version_info`](source/version_info/README.md) — version strings.
2. [`source/main/utils/memory`](source/main/utils/memory/README.md) — reference counting.
3. [`source/main/threadpool`](source/main/threadpool/README.md) — worker pool.
4. [`source/main`](source/main/README.md), hub only — `Application`, `ForwardDeclarations`, `AppContext`.
5. [`utils`](source/main/utils/README.md) (+ [`bbcode`](source/main/utils/bbcode/README.md)) — strings, config files, input, hashing, platform.
6. [`system`](source/main/system/README.md) — settings (CVars), console, command line.
7. [`resources`](source/main/resources/README.md) — every mod file format; mod cache; content manager.
8. [`physics`](source/main/physics/README.md) (+ [`collision`](source/main/physics/collision/README.md), [`air`](source/main/physics/air/README.md), [`water`](source/main/physics/water/README.md), [`flex`](source/main/physics/flex/README.md)) — the soft-body simulation.
9. [`gameplay`](source/main/gameplay/README.md) — drivetrain, character, AI, races, replay, repair.
10. [`terrain`](source/main/terrain/README.md) — terrain loading, objects, roads, editor.
11. [`gfx`](source/main/gfx/README.md) (+ [`camera`](source/main/gfx/camera/README.md), [`particle`](source/main/gfx/particle/README.md)) — rendering from sim buffers.
12. [`audio`](source/main/audio/README.md) — sound scripts and the voice pool.
13. [`network`](source/main/network/README.md) — RoRnet client, HTTP, OutGauge, Discord.
14. [`scripting`](source/main/scripting/README.md) (+ [`bindings`](source/main/scripting/bindings/README.md)) — the AngelScript host and its API.
15. [`gui`](source/main/gui/README.md) (+ [`panels`](source/main/gui/panels/README.md)) — dashboards, HUDs, every window.
16. [`GameContext`](source/main/GameContext.h.md) and [`main.cpp`](source/main/main.cpp.md) — the message queue and the frame loop, which tie everything together.
17. Optional: [`source/microbenchmarks`](source/microbenchmarks/README.md).

## Cycles

- **The hub.** Every file includes `Application.h`, and `Application.h` names types from every chapter. The recipe breaks the cycle at the hub: its twin is pure vocabulary (states, message types, settings, accessors), so it can be read before the chapters it points to.
- **Physics ↔ gameplay ↔ graphics.** Actors own their engine, AI and graphics companion; gameplay and graphics read actors. The recipe breaks this at the physics→graphics edge: physics calls graphics only in a few places, each noted in its twin, and graphics otherwise reads the sim buffer.
- **GUI manager ↔ panels** and **script engine ↔ `game` API**: owner and owned refer to each other. Read the owned side as leaves.

## What is not here

- **Vendored libraries** under `source/main`: `gui/imgui/` (Dear ImGui, 18 files), `gfx/hydrax/` (water, 46 files) and `gfx/skyx/` (sky, 39 files). They appear as seams in [SYSTEM-REQUIREMENTS](SYSTEM-REQUIREMENTS.md#3-seams).
- **Build plumbing**: `CMakeLists.txt` files (their dependency facts are folded into SYSTEM-REQUIREMENTS), `RoRVersionDef.h.in`, `plugins_d.cfg.in` (debug twin of [`plugins.cfg.in`](source/main/plugins.cfg.in.md)), `icon.rc`, `ror.ico`, `COPYING`, and the in-tree `ReadMe.txt` notes (folded into chapter READMEs).
- **Outside `source/`**: game content and resources (`resources/`, `content/`), translations, docs, tools, CI and packaging. The recipe describes the formats these files use, not their contents.

## Known defects recorded in twins

Twins describe the source as it is. Where the original's behaviour is clearly unintended, the twin says so under **Notes** and states what a rebuild should do instead. Examples: whisper chat sends a malformed payload ([`network/Network.cpp`](source/main/network/Network.cpp.md)); `functionExists` has its result inverted ([`scripting/ScriptEngine.cpp`](source/main/scripting/ScriptEngine.cpp.md)); console message fade-out is broken ([`gui/panels/GUI_ConsoleView.cpp`](source/main/gui/panels/GUI_ConsoleView.cpp.md)); the server-list error path posts the wrong payload type ([`gui/panels/GUI_MultiplayerSelector.cpp`](source/main/gui/panels/GUI_MultiplayerSelector.cpp.md)). Search the twins for "a rebuild" to find the rest.
