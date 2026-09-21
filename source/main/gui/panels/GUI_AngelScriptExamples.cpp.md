# source/main/gui/panels/GUI_AngelScriptExamples.cpp

> Rows of example calls on the player vehicle, executed through the console.

**Needs** — [`GUI_AngelScriptExamples.h`](GUI_AngelScriptExamples.h.md) · [`scripting/ScriptEngine.h`](../../scripting/ScriptEngine.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_AngelScriptExamples.h`](GUI_AngelScriptExamples.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_AngelScriptExamples.h`](GUI_AngelScriptExamples.h.md).

## State

See header.

## `Draw`

Rows (call — argument widget — description), all on `game.getCurrentTruck()`: `scaleTruck(slider 1–1.5)`, name / file name / section config / type (logged), `reset(keep position)`, parking brake / traction control / ABS / beacons / custom particles toggles, node count, `getTotalMass(locked)`, wheel node count, `setMass(slider 1000–10000)`, brake light, `getCustomLightVisible(n)`, `setCustomLightVisible(n, on)`, beacon mode, `setBlinkType(n)`, blink type, custom particle mode, reverse light, heading angle, `isLocked`, wheel speed, speed, g-forces, rotation, position, node position. Values are logged with `game.log`.

## `ExecuteString(code)`

Runs `as <code>` through the console command processor (so it lands in the terrain script's context and its output in the console).

**Notes** — the node-position row substitutes three unused x/y/z members instead of the chosen node number, so it always queries the same node; a rebuild uses the node field.
