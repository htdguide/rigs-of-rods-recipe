# System requirements

What the world must provide before a line of Rigs of Rods is rebuilt. Read this before choosing a language or installing anything.

## 1. What this builds

Rigs of Rods (RoR) is a single-player and multiplayer desktop sandbox in which vehicles — cars, trucks, trains, boats, aircraft, cranes — are simulated as **soft bodies**: a mass-spring network of point masses (nodes) joined by damped springs (beams), stepped at 2 kHz. Vehicles are not rigid meshes; they bend, break and deform because the network does. A player walks around a terrain as a character, spawns vehicles from a library of user-made mods, drives them with keyboard, gamepad or wheel, and optionally joins a server where other players' vehicles are replicated. Mods (vehicles, terrains, skins, add-on parts, scripts) are plain-text definition files plus meshes and textures inside zip archives; a large part of the program exists to discover, cache, parse and validate them.

## 2. Tier

| Tier | Name | The repo demands | Typically |
|---|---|---|---|
| **T0** | Metal | No runtime, no allocator you did not write; bytes at fixed addresses; interrupts, MMIO, boot | asm, C, Zig, Rust `no_std` |
| **T1** | Manual | Deterministic destruction, explicit layout, hard latency or memory budgets, FFI as a first-class concern | C, C++, Rust, Zig |
| **T2** | Managed native | Compiled or JIT with a GC; throughput and data layout still matter; concurrency is explicit | Go, Java, C#, Swift, Kotlin |
| **T3** | Dynamic | Iteration speed over throughput; the library ecosystem carries most of the weight | Python, TypeScript, Ruby, Elixir |
| **T4** | Glue | Orchestration, config and text; spawning processes is the primary abstraction | Shell, Make, Nix, small Python |

> **Tier: T2 (Managed native).** Floor constraint: the physics core steps every actor at a fixed 0.5 ms, so a 60 Hz frame runs ~33 substeps, each of which touches every beam (thousands to tens of thousands per vehicle) and every node, then resolves node-vs-triangle collisions between actors. That is tens of millions of small float updates per second with a frame budget of ~16 ms, which rules out any language whose aggregates are boxed by default or whose hot loop is interpreted — T3 is viable only if the whole `physics/` chapter is pushed into a native extension, which is most of the program. T2 works provided node and beam storage are flat arrays of unboxed value records and the substep loop allocates nothing (so the collector never runs mid-frame); C#, Swift, Go and Kotlin/JVM with primitive arrays all qualify. The original is C++ (T1), but nothing in the design needs manual destruction: object lifetimes are reference-counted (`RefCountingObjectPtr`) or owned by long-lived managers, and the one hard deadline is throughput, not deterministic teardown.

A second, softer constraint: the rendering, GUI, audio and scripting seams below are native libraries. A T2 rebuild either binds them (C#, Swift and Kotlin/Native all have mature 3D-engine and OpenAL bindings) or substitutes native-to-the-language equivalents.

## 3. Seams

### Seam: 3D rendering engine

**Verdict** — given

**Must provide**
- A scene graph of nodes with position/orientation/scale, attachable entities, lights and cameras; per-node visibility masks (used to hide objects from mirror/depth passes)
- Loading of a binary mesh format with submeshes, skeletons and skeletal animation (character), plus runtime-created "manual" meshes whose vertex buffers are rewritten every frame (flexbodies, cab triangles, beam lines)
- A material system with named, cloneable materials, per-pass texture units, and script-defined materials loadable from text files (mods ship materials as text)
- Named resource groups over directories and zip archives, with lookup by name across a group and "which group contains resource X" queries
- Render-to-texture targets with their own cameras (mirrors, video cameras, environment cubemap, minimap)
- A 2D overlay layer with text areas and panels (legacy dashboards, loading screen)
- CPU-side particle systems with custom affectors (fire, extinguisher, dust, exhaust)
- Billboards/sets (flares), ribbon trails (skidmarks)
- Heightmap terrain with multi-layer texture blending, paging, and height queries at arbitrary x/z
- Shadow maps with parallel-split (PSSM) cascades, and a runtime shader generator for fixed-function-less backends
- Screenshot capture of the render window to an image file

**Surface used** — broad: roughly 60 engine classes. The load-bearing ones are the scene manager and scene nodes, entity/mesh/submesh, manual hardware vertex/index buffers, material manager, resource group manager, render texture, overlay manager, particle system manager, terrain group, and the log manager (which doubles as RoR's log sink). See [`gfx/`](source/main/gfx/README.md) and [`physics/flex/`](source/main/physics/flex/README.md).

**Known-good substitutes** — Ogre 1.x, as in the original (1.11+; the engine's own mesh and material script formats are what mods ship, which is the strongest argument to stay). Ogre-Next or Godot if a mesh/material converter is written for mod content. A from-scratch Vulkan/WebGPU renderer is possible but must reproduce the material-script and mesh formats.

**If you build it** — not worth it. The engine is larger than the game, and mod content depends on its file formats.

### Seam: Windowing and input devices

**Verdict** — given

**Must provide**
- A resizable window (fullscreen/windowed toggle at runtime) owning a GPU context for the rendering engine
- Keyboard events with press/release and key repeat distinguished, plus text input as UTF-32 code points
- Mouse motion (absolute and relative), buttons, wheel; optional pointer grab
- Joysticks/gamepads/wheels: up to several devices, each with enumerated axes, buttons, POV hats and sliders, polled per frame
- Force feedback: constant-force and spring effects on a wheel, with gain control

**Surface used** — device enumeration, buffered keyboard/mouse listeners, unbuffered joystick state, one force-feedback effect object. See [`utils/InputEngine.cpp`](source/main/utils/InputEngine.cpp.md) and [`utils/ForceFeedback.cpp`](source/main/utils/ForceFeedback.cpp.md).

**Known-good substitutes** — SDL2/SDL3 (window, keyboard, mouse, gamepad and haptics in one); GLFW plus a separate joystick/haptics library; OIS, as in the original.

**If you build it** — not worth it; per-OS device work.

### Seam: Immediate-mode GUI

**Verdict** — given

**Must provide**
- Immediate-mode widgets: windows, buttons, checkboxes, sliders, combo boxes, text input, tables/columns, trees, tabs, menus, tooltips, images from engine textures
- A draw-list output (vertices + indices + texture ids + clip rects) that the rendering engine can submit as an overlay
- Font atlas building from TTF files with custom glyph ranges (for translated UI)
- A per-frame input feed (mouse, keys, text) and "wants mouse / wants keyboard" queries so the game can decide who owns input

**Surface used** — the whole public widget API; almost every panel in [`gui/panels/`](source/main/gui/panels/README.md) is written against it. It is also exposed to scripts ([`scripting/bindings/ImGuiAngelscript.cpp`](source/main/scripting/bindings/ImGuiAngelscript.cpp.md)), so its API shape leaks into mod scripts.

**Known-good substitutes** — Dear ImGui (vendored in the original, together with an engine renderer backend); any immediate-mode library with a draw-list output (Nuklear, egui). Script compatibility requires the ImGui API names.

**If you build it** — not worth it.

### Seam: Positional audio

**Verdict** — given

**Must provide**
- Mono sound sources positioned in 3D with velocity (Doppler), gain, pitch, looping, and a listener with position/velocity/orientation
- A fixed pool of hardware voices smaller than the number of logical sources (the original assumes ~32 hardware sources shared among hundreds of logical ones)
- Loading of Ogg Vorbis and WAV samples into buffers
- Optional environmental reverb (EFX / EAX reverb presets), low-pass filters for occlusion/obstruction, and an auxiliary effect slot per listener environment

**Surface used** — device/context open, buffers, sources, listener, EFX effects/filters/slots. See [`audio/SoundManager.cpp`](source/main/audio/SoundManager.cpp.md).

**Known-good substitutes** — OpenAL Soft, as in the original (only one with EFX); miniaudio or FMOD if reverb is re-implemented; SDL audio + own mixer.

**If you build it** — a basic mixer with distance attenuation and Doppler is a few weeks; EFX-grade reverb is not worth it.

### Seam: Retained layout GUI

**Verdict** — given

**Must provide**
- Load widget trees from content-authored XML layout files (MyGUI `.layout` format), with per-widget free-form user attributes readable by name
- Image boxes whose texture can be swapped at runtime, text boxes, and an image skin that rotates about a pivot
- Render a layer either to screen or into a render-target texture of a given size (for 3D cockpit dashboards)
- Load extra skin/resource definition files (`.resource`) at runtime

**Surface used** — layout load/unload, widget enumeration and user strings, position/size/visibility, texture swap, rotation angle, render-to-texture layers. See [`gui/DashBoardManager.cpp`](source/main/gui/DashBoardManager.cpp.md), [`gui/RTTLayer.cpp`](source/main/gui/RTTLayer.cpp.md).

**Known-good substitutes** — MyGUI, as in the original. Any retained GUI works only if it can read the existing `.layout` files; otherwise ship a converter, since hundreds of community dashboards use this format.

**If you build it** — feasible: the dashboard feature needs only a layout reader (nested widgets with position, size, skin, texture, caption, user strings) and a quad renderer with rotation, render-to-texture included. Weeks, not months.

### Seam: Script engine

**Verdict** — given

**Must provide**
- An embeddable, statically-typed scripting language with C-like syntax that compiles **AngelScript** source text unchanged — mods and terrains ship `.as` files, so the language is part of the content format
- Registration of host types (value types and reference types with ref-counting), methods, properties, global functions, enums and funcdefs
- Script modules that can be built from several sections (includes), discarded, and have functions looked up by declaration
- Contexts that can execute a function with arguments, be suspended/aborted, and report line-level exceptions
- Script arrays, dictionaries, strings, `any`, math — the standard add-ons

**Surface used** — engine creation, ~1,500 registration calls across [`scripting/bindings/`](source/main/scripting/bindings/README.md), module build, context execute, exception and line callbacks.

**Known-good substitutes** — AngelScript 2.3x only. A rebuild in another language must bind the AngelScript C/C++ library or drop script compatibility with existing mods.

**If you build it** — only if you are willing to write an AngelScript compiler; not recommended.

### Seam: HTTP client

**Verdict** — given

**Must provide**
- HTTPS GET with custom headers, following redirects, progress callback with abort, response body to memory or file
- Runs on a worker thread; results are posted back to the main loop as messages

**Surface used** — five call sites: server list, repository list/details, repository file download, thumbnail download, AI-preset download. See [`network/CurlHelpers.cpp`](source/main/network/CurlHelpers.cpp.md).

**Known-good substitutes** — libcurl, as in the original; any language's standard HTTPS client.

**If you build it** — not worth it (TLS).

### Seam: TCP sockets

**Verdict** — given

**Must provide**
- Blocking TCP client connect by hostname and port with timeout, blocking send/receive of exact byte counts, and shutdown from another thread

**Surface used** — connect, send, recv, close. See [`network/Network.cpp`](source/main/network/Network.cpp.md).

**Known-good substitutes** — any OS socket API; SocketW (original).

**If you build it** — trivial over OS sockets.

### Seam: UDP datagrams

**Verdict** — given

**Must provide**
- Send a fixed-size datagram to a configured IPv4 address and port.

**Surface used** — one socket, one send per interval. See [`network/OutGauge.cpp`](source/main/network/OutGauge.cpp.md).

**Known-good substitutes** — OS sockets.

**If you build it** — trivial.

### Seam: JSON

**Verdict** — buildable

**Must provide**
- Parse to a DOM; read object members, arrays, numbers, strings, bools; build a DOM and serialize it (pretty or compact)

**Surface used** — mod cache index, savegames, repository and server-list API responses, waypoints, AI presets, local script storage.

**If you build it** — any standard JSON library; do not write one.

### Seam: Zip archives

**Verdict** — given

**Must provide**
- Read-only listing and streaming of entries in `.zip` files, used as mounted directories by the resource system
- Stable per-archive hashing input (file size + modification time) for cache invalidation

**Surface used** — indirectly through the rendering engine's archive layer.

**Known-good substitutes** — zziplib/minizip/libzip; any language's stdlib zip.

**If you build it** — not worth it.

### Seam: Sky rendering

**Verdict** — pluggable

**Must provide** — a sky that can be (a) a static skybox from terrain config, or (b) a dynamic day/night sky with sun/moon and clouds, exposing: time of day get/set, time speed, sun direction (for shadows and water), fog/ambient colors, and an update per frame with camera. RoR's interface is `SkyManager` / `SkyXManager` ([`gfx/`](source/main/gfx/README.md)).

**Known-good substitutes** — Caelum (optional in original), SkyX (vendored in original); a shader-based Preetham/Hosek sky.

### Seam: Water rendering

**Verdict** — pluggable

**Must provide** — the interface `IGfxWater` ([`gfx/IGfxWater.h`](source/main/gfx/IGfxWater.h.md)): water plane at a height, visibility, reflection/refraction targets, per-frame update with camera. The physical wave height comes from the separate, repo-owned [`physics/water/Wavefield`](source/main/physics/water/Wavefield.cpp.md), not from the renderer.

**Known-good substitutes** — the built-in `GfxWater` (plane + RTT reflection/refraction); Hydrax (vendored in original; FFT/Perlin ocean).

### Seam: Vegetation paging

**Verdict** — given (optional)

**Must provide** — instanced/billboarded trees and grass placed by density maps, paged by camera distance, with LODs and collision proxies for trees.

**Known-good substitutes** — PagedGeometry (original); engine-native instancing + impostors.

### Seam: Localization catalogs

**Verdict** — buildable

**Must provide** — load a GNU gettext `.mo` catalog for the chosen language and translate `msgid` → `msgstr` at runtime.

**If you build it** — ~150 lines: parse the `.mo` header, two string tables, binary-search or hash. The original uses the header-only moFileReader.

### Seam: Discord rich presence

**Verdict** — given (optional)

**Must provide** — set a "now playing" status string with a terrain/vehicle name and start time.

**Known-good substitutes** — Discord Game SDK; discord-rpc (original).

### Seam: Mumble positional link

**Verdict** — buildable

**Must provide** — write the avatar and camera position/orientation into the Mumble "Link" shared-memory struct each frame so voice chat is positional.

**If you build it** — map a named shared-memory region (`MumbleLink` on Windows, `/MumbleLink.<uid>` on POSIX) and fill a documented fixed-layout struct; see [`audio/MumbleIntegration.cpp`](source/main/audio/MumbleIntegration.cpp.md).

### Seam: Thread pool

**Verdict** — buildable

**Must provide** — a fixed set of worker threads that run submitted closures, a join handle per task, and a "run all these and wait" primitive.

**If you build it** — see [`threadpool/ThreadPool.h`](source/main/threadpool/ThreadPool.h.md); ~200 lines on any OS thread + mutex + condition-variable API. Most T2 languages have one built in.

### Seam: Text formatting

**Verdict** — buildable

**Must provide** — `printf`-style and `{}`-style string formatting with numeric precision. Any language's standard formatter.

## 4. Platform assumptions

- **OS**: Windows 10+ and Linux (X11) on x86-64; a few macOS code paths exist but macOS is not a maintained target. User directory (verified in [`AppContext.cpp`](source/main/AppContext.cpp.md)): a `config` folder next to the executable if it exists (portable install); otherwise `<home>/My Games/Rigs of Rods` on Windows, `$SNAP_USER_COMMON` or `~/.rigsofrods` on Linux, `~/RigsOfRods` on macOS. Subfolders: `config`, `cache`, `logs`, `thumbnails`, `savegames`, `screenshots`, `scripts`, `projects`, `repo_attachments`, `mods` ([`main.cpp`](source/main/main.cpp.md#startup-order-matters)).
- **Filesystem**: case sensitivity differs by OS; mod file lookups go through the resource system, which is case-insensitive on Windows and case-sensitive on Linux. Mod authors rely on Windows behaviour, so a rebuild should do case-insensitive resource lookup everywhere.
- **Byte order**: little-endian is assumed for the network protocol and binary caches (packed structs are sent raw).
- **Word size**: 64-bit; `int` is 32-bit in all wire formats.
- **Floating point**: single precision (`float`) throughout physics; results are not bit-reproducible across machines and nothing depends on that, except replays, which are recorded state, not re-simulated.
- **Threads**: real OS threads. The main thread owns rendering, GUI and the message queue; physics can run on one dedicated thread concurrently with rendering (see [`physics/ActorManager.cpp`](source/main/physics/ActorManager.cpp.md)); a worker pool (sized to logical cores) parallelizes per-actor substeps and inter-actor collision; networking runs a send thread and a receive thread; HTTP requests run on pool workers.
- **Clocks**: a monotonic millisecond clock for frame timing and a wall clock for timestamps and chat.
- **Locale**: all text is UTF-8; numeric parsing in content files must be locale-independent (`.` decimal separator).
- **Network**: optional. Offline play is fully supported; online features (multiplayer, server list, repository browser, AI preset download) simply fail with a message when unreachable, and the race-results API can be switched off by a config flag (`app_disable_online_api`).

## 5. Data and persistence

**Must match exactly** (someone else already holds these files or peers):

- **Truck definition family** — `.truck .car .boat .airplane .train .trailer .load .fixed .machine` (all nine registered as known extensions in the mod cache): line-oriented, keyword-sectioned text. Tens of thousands of community mods exist. Grammar and semantics: [`resources/rig_def_fileformat/`](source/main/resources/rig_def_fileformat/README.md).
- **Add-on parts and tune-ups** — `.addonpart`, `.tuneup`: see [`resources/addonpart_fileformat/`](source/main/resources/addonpart_fileformat/README.md) and [`resources/tuneup_fileformat/`](source/main/resources/tuneup_fileformat/README.md).
- **Skins** — `.skin`: [`resources/skin_fileformat/`](source/main/resources/skin_fileformat/README.md).
- **Terrains** — `.terrn2` (INI), `.otc` (Ogre terrain config), `.tobj` (object placement), `.odef` (object definition): [`resources/`](source/main/resources/README.md) sub-chapters.
- **Sound scripts** — `.soundscript`: [`audio/SoundScriptManager.cpp`](source/main/audio/SoundScriptManager.cpp.md).
- **Ground models** — `ground_models.cfg`: [`physics/collision/Collisions.cpp`](source/main/physics/collision/Collisions.cpp.md).
- **Torque curves, inertia models** — `torque_models.cfg`, `inertia_models.cfg`.
- **Input mapping** — `input.map` and per-device `*.map`: [`utils/InputEngine.cpp`](source/main/utils/InputEngine.cpp.md).
- **Game settings** — `RoR.cfg` (key=value): [`system/AppConfig.cpp`](source/main/system/AppConfig.cpp.md).
- **Scripts** — AngelScript `.as` against the registered API: [`scripting/`](source/main/scripting/README.md).
- **RoRnet 2.45** — multiplayer wire protocol, little-endian packed structs, version string `RoRnet_2.45`: [`network/RoRnet.h`](source/main/network/RoRnet.h.md). Servers exist in the wild; any change breaks interop.
- **OutGauge** — the Live-for-Speed telemetry UDP packet, 96 bytes including the trailing ID (the game always sends the ID); Windows builds only: [`network/OutGauge.cpp`](source/main/network/OutGauge.cpp.md).
- **Rendering-engine formats** — `.mesh`, `.skeleton`, `.material`, `.particle`, `.overlay`, `.compositor`; mods ship these.
- **Dashboards** — `.dashboard` mods: retained-GUI `.layout` files whose widgets carry user attributes `anim[N]`, `link[N]` (slot name, optional `>x` / `<x`), `min/max/vmin/vmax[N]`, `texture[N]`, `format[N]`, `direction[N]`, `debug`, plus `dashboard_custom_input` lines, file-name tags `<N>rpm` / `kph` / `mph`, `*.resource` skins and an optional `<name>.as` script: [`gui/DashBoardManager.cpp`](source/main/gui/DashBoardManager.cpp.md). Slot names are listed in [`gui/DashBoardManager.h`](source/main/gui/DashBoardManager.h.md).
- **Chat colour marks** — `#RRGGBB` inside chat text and player names: [`gui/GUIUtils.cpp`](source/main/gui/GUIUtils.cpp.md#colour-marked-text).
- **Multiplayer server list** — `GET <mp_api_url>/server-list?json=true`, a JSON array of `{name, terrain-name, ip, port, has-password, current-users, max-clients, version}`: [`gui/panels/GUI_MultiplayerSelector.cpp`](source/main/gui/panels/GUI_MultiplayerSelector.cpp.md).
- **Repository web API** — `<remote_query_url>/resources`, `/resource-categories`, `/resources/<id>` (JSON), file downloads `https://forum.rigsofrods.org/resources/<id>/download?file=<file id>`, attachments `…/attachments/<id>`; descriptions are BBCode: [`gui/panels/GUI_RepositorySelector.cpp`](source/main/gui/panels/GUI_RepositorySelector.cpp.md). Read-only consumer.
- **Online API (race results)** — JSON `POST <mp_api_url><query>` with `RoR-Api-User` / `RoR-Api-User-Token` headers: [`scripting/GameScript.cpp`](source/main/scripting/GameScript.cpp.md#useonlineapi).
- **AI waypoint presets** — JSON array of `{terrain, preset, waypoints: [[x, y, z(, speed)]]}` (and `{terrains: [...]}` index entries), bundled via `.terrn2 [AI Presets]`, from `savegames/waypoints.json`, or downloaded from the community repository: [`gui/panels/GUI_TopMenubar.cpp`](source/main/gui/panels/GUI_TopMenubar.cpp.md#ai-preset-sources).
- **Script API names** — every type, method and enum registered for scripts: [`scripting/bindings/`](source/main/scripting/bindings/README.md).

**Internal, choose freely:**

- The mod cache index (`mods.cache` in the cache folder, JSON; verified in [`resources/CacheSystem.cpp`](source/main/resources/CacheSystem.cpp.md)) — regenerated when absent or on version bump.
- Savegames (JSON, versioned) — only this program reads them; keep them readable across your own versions.
- Replays — in-memory only.
- Local script storage (`*.asdata`) — script-private key/value files.
- Log files.

## 6. Conformance

RoR has no unit-test suite. What exists, and what a rebuild should reproduce:

- **Parser benchmark** — [`source/microbenchmarks/`](source/microbenchmarks/README.md) times keyword-identification strategies for the truck parser. It checks no results; its only conformance content is the keyword list (105 section keywords). Keyword precedence rules live in the parser itself ([`RigDef_Parser`](source/main/resources/rig_def_fileformat/RigDef_Parser.cpp.md)).
- **Round-trip** — a truck file parsed ([`RigDef_Parser`](source/main/resources/rig_def_fileformat/RigDef_Parser.cpp.md)) and serialized ([`RigDef_Serializer`](source/main/resources/rig_def_fileformat/RigDef_Serializer.cpp.md)) must re-parse to the same document; an actor exported ([`ActorExport.cpp`](source/main/physics/ActorExport.cpp.md)) must spawn identically.
- **Validator** — the rules in [`RigDef_Validator`](source/main/resources/rig_def_fileformat/RigDef_Validator.cpp.md) (required sections, engine requires `engoption`/wheels, etc.) are the acceptance gate for a vehicle.
- **Physics invariants checked at runtime** — beams break when stress exceeds `strength`; deformation is plastic beyond `deform` threshold; the actor manager clamps a frame to at most 1/20 s of simulated time and carries the fractional remainder; a NaN node position resets the actor ([`physics/Actor.cpp`](source/main/physics/Actor.cpp.md)).
- **Protocol** — connect to a RoRnet 2.45 server (the public server software is the oracle): HELLO (client version) → HELLO (server info) → USER_INFO → WELCOME → stream registration, chat round-trip, and a remote actor appearing on another client.
- **Content oracle** — the base content pack and the community's official content packs must load without validator errors and drive plausibly; this is the practical acceptance test RoR's own developers use.
- **Script API** — every function listed in `doc/angelscript` (upstream docs) must be callable from a script; the example scripts in `resources/scripts/` must compile.

## 7. Build order

Chapters in dependency order, leaves first. Nearly every chapter includes the hub header `Application.h`, and physics, gameplay and graphics include each other; the cycle is broken at the hub (its twin is pure vocabulary), and at the physics→graphics edge (physics only pokes graphics through a handful of calls noted in each twin).

1. [`source/version_info`](source/version_info/README.md) — build version strings.
2. [`source/main/utils/memory`](source/main/utils/memory/README.md) — intrusive reference counting.
3. [`source/main/threadpool`](source/main/threadpool/README.md) — worker pool.
4. [`source/main`](source/main/README.md) — the hub: global enums, CVars, message types, object registry (`Application`), render-window context (`AppContext`). `GameContext` and `main.cpp` live here but are the **last** things to read.
5. [`source/main/utils`](source/main/utils/README.md) and [`utils/bbcode`](source/main/utils/bbcode/README.md) — strings, config files, input engine, hashing, platform paths.
6. [`source/main/system`](source/main/system/README.md) — CVars, console, config, command line.
7. [`source/main/resources`](source/main/resources/README.md) and its format sub-chapters — every mod file format, the mod cache, content manager.
8. [`source/main/physics`](source/main/physics/README.md) with [`collision`](source/main/physics/collision/README.md), [`air`](source/main/physics/air/README.md), [`water`](source/main/physics/water/README.md), [`flex`](source/main/physics/flex/README.md) — the soft-body simulation.
9. [`source/main/gameplay`](source/main/gameplay/README.md) — engine/drivetrain, character, AI, races, replay.
10. [`source/main/terrain`](source/main/terrain/README.md) — terrain loading, objects, roads, editor.
11. [`source/main/gfx`](source/main/gfx/README.md) with [`camera`](source/main/gfx/camera/README.md), [`particle`](source/main/gfx/particle/README.md) — visual representation.
12. [`source/main/audio`](source/main/audio/README.md) — sound scripts, voice pool.
13. [`source/main/network`](source/main/network/README.md) — multiplayer, HTTP, telemetry.
14. [`source/main/scripting`](source/main/scripting/README.md) with [`bindings`](source/main/scripting/bindings/README.md) — AngelScript host.
15. [`source/main/gui`](source/main/gui/README.md) with [`panels`](source/main/gui/panels/README.md) — UI.
16. [`source/microbenchmarks`](source/microbenchmarks/README.md) — a stand-alone benchmark behind a truck-parser design choice (optional).
17. Return to [`source/main/GameContext.cpp`](source/main/GameContext.cpp.md) and [`source/main/main.cpp`](source/main/main.cpp.md) — the conductor.
