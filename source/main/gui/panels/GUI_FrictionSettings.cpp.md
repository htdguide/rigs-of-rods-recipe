# source/main/gui/panels/GUI_FrictionSettings.cpp

> Sliders over a ground model working copy; changes are applied through the message queue.

**Needs** — [`GUI_FrictionSettings.h`](GUI_FrictionSettings.h.md) · [`Application.h`](../../Application.h.md) · [`physics/SimData.h`](../../physics/SimData.h.md) · [`physics/collision/Collisions.h`](../../physics/collision/Collisions.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`terrain/Terrain.h`](../../terrain/Terrain.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_FrictionSettings.h`](GUI_FrictionSettings.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_FrictionSettings.h`](GUI_FrictionSettings.h.md). Parameters are defined in [`physics/collision/Collisions`](../../physics/collision/Collisions.h.md).

## State

See header.

## `Draw`

Shows the ground under the player, a combo of all ground models, and sliders on the selected model's working copy, each with a `[?]` tooltip explaining it:

| Group | Parameter | Range |
|---|---|---|
| Solid | solid ground level | 0–200 |
| | strength | 0–2 |
| | static friction coef (ms) | 0.1–2 |
| | adhesion velocity (va) | 0.1–5 |
| | dynamic friction coef (mc) | 0.1–1.5 |
| | hydrodynamic friction coef (t2) | 0–1.5 |
| | Stribeck velocity (vs) | 0–1000 |
| | alpha | 0–200 |
| Fluid | flow behaviour index | −2–2 |
| | flow consistency | 10–100000 |
| | fluid density | 10–100000 |
| | drag anisotropy | 0–1 |

Any change posts "modify ground model" carrying the working copy; the physics side copies it into the live model at a safe point.

**Notes** — the backup copy is kept but no "revert" exists.
