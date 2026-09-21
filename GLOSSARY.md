# Glossary

Domain terms used across the recipe, defined once so the twins can stay terse. Terms are grouped by topic; each entry links to the twin that owns the concept.

## Simulation

- **Actor** — one simulated soft body: a vehicle, load, trailer, machine or fixed structure spawned from a truck file. Owned by the actor manager; also called "truck" or "vehicle" in older names. [`physics/Actor.h`](source/main/physics/Actor.h.md)
- **Node** — a point mass in an actor. It has position, velocity, accumulated force, mass and options (for example, "loaded" or "contactless"). [`physics/SimData.h`](source/main/physics/SimData.h.md)
- **Beam** — a spring-damper between two nodes. It can deform plastically and break when stress exceeds its strength. Special kinds are listed below. [`physics/SimData.h`](source/main/physics/SimData.h.md)
- **Shock** — a beam with separate spring and damping for extension and compression (`shocks`, `shocks2`, `shocks3`).
- **Hydro** — a beam whose rest length follows steering or another input (steering racks, hydraulic arms).
- **Command beam** — a beam whose length is driven by a **command key** (1…84) that the player presses; `commands2` beams have separate extend and retract rates.
- **Rope, tie, hook** — beams created at runtime to link an actor to another actor or to the world. A **rope** has fixed length. A **tie** tightens around a load ("secure load"). A **hook** locks a node to a node on another actor ("lock"). Linked actors form a group that is teleported, reset and paused together.
- **Slide node** — a node constrained to slide along a **rail** of beams. It may lock to rails on other actors. [`physics/SlideNode.h`](source/main/physics/SlideNode.h.md)
- **Wheel** — a ring of nodes around an axle, with the beams that make up the tyre; types are `wheels`, `wheels2`, `meshwheels`, `meshwheels2`, `flexbodywheels`. [`physics/SimData.h`](source/main/physics/SimData.h.md)
- **Cinecam** — a node set that acts as the driver's head position; entering a vehicle requires one.
- **Physics step** — one 0.5 ms integration step (2 kHz). Each frame runs as many steps as fit in the frame's time, clamped to 1/20 s, and carries the remainder over. [`physics/ActorManager.cpp`](source/main/physics/ActorManager.cpp.md)
- **Sim buffer** — a snapshot of the simulation state that the main thread copies once per frame, while the physics thread is synced. Graphics and most of the GUI read only this snapshot, so rendering can overlap the next physics step. [`gfx/GfxActor.h`](source/main/gfx/GfxActor.h.md)
- **Actor state** — `LOCAL_SIMULATED`, `LOCAL_SLEEPING` (at rest, not stepped), `LOCAL_REPLAY`, `NETWORKED_OK`, `NETWORKED_HIDDEN`.
- **Ground model** — a set of friction and fluid parameters for a surface type, such as asphalt, mud or sand. [`physics/collision/Collisions.h`](source/main/physics/collision/Collisions.h.md)
- **Landuse map** — an image that selects a ground model per terrain position. [`gameplay/Landusemap.h`](source/main/gameplay/Landusemap.h.md)
- **Collision box / event box** — an oriented box on the terrain. A solid box collides. A virtual **event box** only fires script events when nodes or characters enter or leave it; spawn boxes and repair areas are event boxes. [`physics/collision/Collisions.h`](source/main/physics/collision/Collisions.h.md)
- **Free force** — a scriptable force applied to one node. It can be constant, toward a point, toward another node, or a "half-beam" spring. [`physics/ActorManager.h`](source/main/physics/ActorManager.h.md)
- **Soft reset / reset on spot / live repair** — ways of restoring an actor. A soft reset restores nodes to their spawn shape where the actor stands. Reset on spot keeps the current position. Live repair is an interactive mode for moving and rotating the actor while it is being restored. [`gameplay/RepairMode.h`](source/main/gameplay/RepairMode.h.md)

## Content

- **Truck file** — the text definition of an actor (`.truck .car .boat .airplane .train .trailer .load .fixed .machine`), organised in keyword sections. [`resources/rig_def_fileformat/`](source/main/resources/rig_def_fileformat/README.md)
- **Section config** — a named variant inside a truck file that includes or excludes `section` blocks.
- **Flexbody** — a mesh that deforms with the node cloud. Each vertex is attached to a reference node and two axis nodes. [`physics/flex/FlexBody.h`](source/main/physics/flex/FlexBody.h.md)
- **Prop** — a rigid mesh attached to three nodes, such as a dashboard, steering wheel, beacon or mirror. [`gfx/GfxData.h`](source/main/gfx/GfxData.h.md)
- **Flare** — a light source on an actor: head, brake, reverse, blinker, fog, side, user-numbered, or dashboard-driven.
- **Managed material** — a material that the actor rewrites, for example for damage textures or mirrors.
- **Skin** — a `.skin` file that replaces textures and materials of a vehicle, matched by GUID. [`resources/skin_fileformat/`](source/main/resources/skin_fileformat/README.md)
- **Add-on part / tune-up** — `.addonpart` adds or replaces elements of a vehicle. `.tuneup` is a saved selection of add-on parts and tweaks (force-removed or protected elements, wheel sides, camera roles). [`resources/addonpart_fileformat/`](source/main/resources/addonpart_fileformat/README.md), [`resources/tuneup_fileformat/`](source/main/resources/tuneup_fileformat/README.md)
- **Terrain files** — `.terrn2` is the terrain descriptor, `.otc` the height-map geometry config, `.tobj` the object placements and procedural roads, and `.odef` the object definition (mesh plus collision boxes and meshes). [`resources/`](source/main/resources/README.md)
- **Asset pack** — a bundle of shared meshes and textures that terrains or vehicles declare they need.
- **Gadget** — a `.gadget` mod that wraps a script and its assets, launched from the tools menu.
- **Dashboard** — a data-driven instrument layout (`.dashboard` mod with `.layout` files) animated from dashboard data slots. [`gui/DashBoardManager.h`](source/main/gui/DashBoardManager.h.md)
- **Sound script** — a `.soundscript` template that maps triggers and modulation sources to sounds with pitch and gain curves. [`audio/SoundScriptManager.h`](source/main/audio/SoundScriptManager.h.md)
- **GUID / UID** — a GUID identifies a vehicle family, and skins and add-on parts match it. A UID prefix (`1234UID-name.truck`) identifies a repository upload.

## Content management

- **Resource group** — a named set of locations searched together when a file is opened by name. Every loaded mod bundle gets its own group. [`resources/ContentManager.h`](source/main/resources/ContentManager.h.md)
- **Resource bundle** — a zip archive or directory that holds one or more mods.
- **Mod cache / cache entry** — the index of all installed content (`mods.cache`). One entry per loadable file records its type, names, authors, GUID, category, counts and bundle. Bundles load lazily. [`resources/CacheSystem.h`](source/main/resources/CacheSystem.h.md)
- **Loader type** — the content kind used by the cache and the selector: terrain, truck, car, boat, airplane, trailer, train, load, extension, skin, add-on part, tune-up, asset pack, dashboard, gadget.
- **Project** — a writable copy of a mod in the user's projects folder. Tuning and node/beam tuning save into projects.

## Engine infrastructure

- **CVar** — a named, typed setting (for example `gfx_fps_limit`). It can be set from `RoR.cfg`, the command line, the console or scripts. [`system/CVar.h`](source/main/system/CVar.h.md)
- **Message / message queue / chain** — every structural change is a posted message that the main loop applies at a safe point. A **chain** holds messages that must run after their parent. [`GameContext.h`](source/main/GameContext.h.md)
- **App state / sim state** — application: main menu, simulation, shutdown. Simulation: off, running, paused, editor mode.
- **Input event** — a named action (`EV_TRUCK_ACCELERATE`) bound to keys, mouse or joystick in `.map` files; **EXPL** marks an exact key combination. [`utils/InputEngine.h`](source/main/utils/InputEngine.h.md)
- **RTT** — render-to-texture; used for 3D dashboards, mirrors (video cameras) and the survey map.
- **Survey map** — the overview map (mini-map or full-screen). [`gui/panels/GUI_SurveyMap.h`](source/main/gui/panels/GUI_SurveyMap.h.md)
- **Colour mark** — `#RRGGBB` inside text that changes the colour of the text after it. It is used in chat and names. [`gui/GUIUtils.cpp`](source/main/gui/GUIUtils.cpp.md#colour-marked-text)

## Scripting

- **Script unit** — one loaded script, with its own module and globals. Its category is terrain, actor, gadget or custom. [`scripting/ScriptEngine.h`](source/main/scripting/ScriptEngine.h.md)
- **Script event** — a notification bit such as `SE_TRUCK_ENTER`, delivered to units that registered for it through `eventCallbackEx`. [`gameplay/ScriptEvents.h`](source/main/gameplay/ScriptEvents.h.md)
- **Bindings** — the script-visible API names, a compatibility contract with existing mods. [`scripting/bindings/`](source/main/scripting/bindings/README.md)

## Multiplayer

- **RoRnet** — the multiplayer wire protocol (version 2.45). [`network/RoRnet.h`](source/main/network/RoRnet.h.md)
- **Stream** — a per-user channel for one networked object: an actor, the character, or chat. Each is registered, then sent as data packets. Actor streams carry vehicle state plus compressed node positions.
- **Peer options** — local-only mute and hide settings per remote player.
- **Net quality** — the server's report that this client is falling behind.

## Gameplay

- **Race** — a checkpoint sequence defined by a terrain script, with timer, best lap and the direction arrow. [`gameplay/RaceSystem.h`](source/main/gameplay/RaceSystem.h.md)
- **Vehicle AI** — waypoint-following driving through `AI.as` and the vehicle AI object. Its modes are normal, race, drag race, crash and chase. [`gameplay/VehicleAI.h`](source/main/gameplay/VehicleAI.h.md)
- **Replay** — a ring buffer of recent node states that the player can scrub. [`gameplay/Replay.h`](source/main/gameplay/Replay.h.md)
- **Character** — the player's avatar on foot, with simplified physics. It can be seated in an actor. [`gameplay/Character.h`](source/main/gameplay/Character.h.md)
