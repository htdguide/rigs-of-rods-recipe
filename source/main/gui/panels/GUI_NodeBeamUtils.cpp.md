# source/main/gui/panels/GUI_NodeBeamUtils.cpp

> Mass and spring/damping sliders, search control, and the create-project path for read-only mods.

**Needs** — [`GUI_NodeBeamUtils.h`](GUI_NodeBeamUtils.h.md) · [`Application.h`](../../Application.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_NodeBeamUtils.h`](GUI_NodeBeamUtils.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_NodeBeamUtils.h`](GUI_NodeBeamUtils.h.md).

## State

See header.

## `Draw`

Closes itself without a player vehicle. 600×675. Drawn only while the simulation is synced (it edits live actor fields).

- Vehicle from a zip (read-only): a banner *Create writable project* — posts a create-project request named `nbutil_<file>` from this vehicle, then chains a "project created, load it" message box and deletes the vehicle; closes the window.
- Vehicle from a project: a menu bar with the project name and *Save and reload* (write the tuned values back into the truck file and respawn).

While searching, `searchBeamDefaults` runs on the actor each frame.

## Mass tab

Dry mass and load mass sliders (0.4–1.6 × original, kg) with reset buttons; minimum node mass scale 0.4–1.6 applied to every node's original minimum; each change recalculates node masses. Shows total mass, node count, loaded node count.

## Spring/Damp tab

- Scale sliders 0.1–10 for beams, shocks and wheels (spring and damping each), plus the wheels' base spring and damping; any change re-applies scales. Shows effective wheel spring/damping.
- *Reset to default settings* (all scales 1, full reset), *Update initial node positions*.
- Search settings: physics steps to skip (0–2000) and to measure (2–6000); lower/upper search bounds for spring and damping of beams, shocks and wheels (0.1–10, lower ≤ upper).
- *Start / Continue / Stop searching* (stopping resets the vehicle), *Reset search*.
- Results when available: reference and optimum movement (per node), stress and jitter (per beam), with totals.
