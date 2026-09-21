# source/main/gui/panels/GUI_FlexbodyDebug.cpp

> Element picker, overlays drawn from the node snapshot, locator table and memory-order graph.

**Needs** — [`GUI_FlexbodyDebug.h`](GUI_FlexbodyDebug.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`Application.h`](../../Application.h.md) · [`physics/SimData.h`](../../physics/SimData.h.md) · [`physics/collision/Collisions.h`](../../physics/collision/Collisions.h.md) · [`physics/flex/FlexBody.h`](../../physics/flex/FlexBody.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`terrain/Terrain.h`](../../terrain/Terrain.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_FlexbodyDebug.h`](GUI_FlexbodyDebug.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_FlexbodyDebug.h`](GUI_FlexbodyDebug.h.md).

## State

See header.

## `AnalyzeFlexbodies`

List flexbodies as `<mesh> (<verts> verts -> <nodes> nodes)` (placeholders by kind) and props by mesh name; special props: aerial nav lights `L/R/w`, dashboard + steering wheel pairs, corrupted props.

## `Draw`

"You are on foot" / "no flexbodies or props" when applicable. Otherwise: element combo; *Hide other* (hides every other mesh — also pauses reflections); mesh name; per-material wireframe toggle; base nodes (ref, x, y) with show toggle; for flexbodies, forset node count, vertex count with *Show all (pick with mouse)*, hovered vertex, a locator table (vertex, ref/x/y nodes, show toggle), and the memory graph; mesh info (original and live for flexbodies; loaded meshes for props).

## Overlay (screen space, from the node snapshot)

Three layers (beams, nodes, text). Base nodes as orange dots with the ref→x axis red and ref→y axis blue. Forset nodes as yellow dots with ids. Vertices as cyan dots; the vertex nearest the mouse within 25 px is "hovered". For every vertex whose locator is shown (or hovered): its label, its three locator nodes, the red/blue axes and a green line from ref node to vertex; clicking a hovered vertex toggles its locator.

## Memory-order graph

Plots, for each vertex left→right, the indices of its ref/x/y nodes bottom→top; ascending dots mean cache-friendly order. Controls for the flexbody "defrag" settings (enable, reorder texcoords, reorder indices, invert lookup; constant, upward and downward penalties 0–15) and *Reload vehicle* to apply them.
