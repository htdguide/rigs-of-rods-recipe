# source/main/gameplay/SceneMouse.cpp

> Ray picking of nodes and actors, the drag force, and click selection.

**Needs** — [`SceneMouse.h`](SceneMouse.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`Application.h`](../Application.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md)
**Used by** — callers of [`SceneMouse.h`](SceneMouse.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`SceneMouse.h`](SceneMouse.h.md).

## State

See header.

## Grabbing

- **Left button pressed while idle (on mouse move)** — cast the camera ray; among simulated local actors whose bounding box the ray hits, find the node (not flagged no-grab) whose 0.1 m sphere is hit nearest. If found, grab it; if that node is a hook node, queue a mouse-hook toggle for its group.
- **Each frame while grabbing** — target = ray point at the grab distance; `actor.mouseMove(node, target, 30000)` (the actor scales the force by `(mass/3000)^0.75` and applies it toward the target during steps).
- **Release** — remove the force and reset (ignored while paused).
- The pick line is drawn from the node's snapshot position to the target.

## Middle click (not paused, not in editor)

- Selects another vehicle: of all actors other than the current one whose minimum-camera-radius sphere the ray hits, the one whose centre is closest to the ray line → request to seat the player in it.
- Sets the orbit camera centre: in vehicle camera mode, the player actor's node (0.25 m spheres) closest to the ray line (ties → nearer to the camera) becomes the custom camera node; the camera context is reset. Clicking empty space sets "none" (back to the default centre).

The mouse ray is built from the last absolute mouse position normalised by the viewport size.
