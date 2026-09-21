# source/main/gfx/GfxScene.cpp

> Per-frame scene update order, particle pools, free beams, net labels, and the stable rotation helper.

**Needs** — [`GfxScene.h`](GfxScene.h.md) · [`AppContext.h`](../AppContext.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`physics/ApproxMath.h`](../physics/ApproxMath.h.md) · [`system/Console.h`](../system/Console.h.md) · [`DustPool.h`](DustPool.h.md) · [`HydraxWater.h`](HydraxWater.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`gui/GUIUtils.h`](../gui/GUIUtils.h.md) · [`gui/panels/GUI_DirectionArrow.h`](../gui/panels/GUI_DirectionArrow.h.md) · [`gui/OverlayWrapper.h`](../gui/OverlayWrapper.h.md) · [`SkyManager.h`](SkyManager.h.md) · [`SkyXManager.h`](SkyXManager.h.md) · [`terrain/TerrainGeometryManager.h`](../terrain/TerrainGeometryManager.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`terrain/TerrainObjectManager.h`](../terrain/TerrainObjectManager.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [Seam: Immediate-mode GUI](../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GfxScene.h`](GfxScene.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GfxScene.h`](GfxScene.h.md).

## State

See header.

## `BufferSimulationData`

Copies player actor, character position, pause, simulation speed, camera behaviour, race timing/arrow into the game-context buffer. Actor visuals are snapshotted only when the actor is *live* (simulated/networked/replaying — state before LOCAL_SLEEPING) or not yet initialised (first frame); those form this frame's live list. Characters always snapshot.

## `UpdateScene(dt)`

```text
queue flexbody and wheel-mesh deformation for live actors (worker threads)
FOV = internal (cinecam) or external setting, unless the camera is static
particles (particle mode 1): per actor (dt = 0 when that actor is paused), then advance all pools
environment map around the player's actor
terrain objects; main-light position
water: reflection plane at the wave height under the player's actor (or static level); step water
sky updates (Caelum detection, SkyX)
race overlay show/hide/update; tyre-pressure overlay
net labels, characters
per actor (dt = 0 when paused): live ones update rods, cab, wings, airbrakes, custom particles, exhausts, aero engines, prop animations;
  all update props and flares
player's actor: video cameras; truck HUD or aircraft HUD
GUI draw (buffered), scene-mouse visuals, free beams
join deformation tasks and upload (live actors)
```

Doing the expensive deformation in parallel with the rest of the frame is the reason for the queue-then-join structure.

## Dust pools

`dust` (tracks/Dust, 20), `clump` (20), `sparks` (10), `drip` (50), `splash` (20), `ripple` (20) — capacities per pool. `AdjustParticleSystemTimeFactor` makes particle systems run at simulation speed, frozen when paused or not running.

## Free beams

Scripts pair up to two half-beam free forces with a visual beam: add (duplicate id → warning), modify (= remove + add; warnings use a literal "%d" — the id is not substituted), remove, and per-frame placement between the base and target nodes of the primary force (a stretched beam mesh like rods). When the secondary force is removed, the beam keeps showing the primary; when the primary is removed or either breaks, the visual is removed.

## `DrawNetLabel(pos, distance, nick, colour)`

Caption "nick (x.y km)" beyond 1 km, "nick (n m)" beyond 20 m, else "nick"; drawn in screen space on a semi-transparent rounded box in the player's colour when in front of the camera.

## `SpecialGetRotationTo(src, dst)`

Shortest-arc quaternion from src to dst, handling the antiparallel case with a 180° turn about an axis perpendicular to src (X × src, or Y × src if colinear) — used to orient beam meshes without flipping.
