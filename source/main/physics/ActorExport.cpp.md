# source/main/physics/ActorExport.cpp

> Writes a live, possibly edited, actor back into its truck document (nodes, beams, cinecams, wheels, shocks, hydros, masses).

**Needs** — [`Actor.h`](Actor.h.md) · [`Application.h`](../Application.h.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`GameContext.h`](../GameContext.h.md) · [`resources/rig_def_fileformat/RigDef_File.h`](../resources/rig_def_fileformat/RigDef_File.h.md)
**Used by** — callers of [`Actor.h`](Actor.h.md) — it implements the actor's export methods
**Tier floor** — T2

## Purpose

Backs the in-game node/beam editing tools: after the user changes spring/damping/positions, `propagateNodeBeamChangesToDef` rewrites the cached document's root module so it can be [serialised](../resources/rig_def_fileformat/RigDef_Serializer.cpp.md) back to a truck file. Declared "proof of concept" by its authors: it assumes a single root module and flattens defaults.

## State

Stateless; mutates the actor's cached document.

## `propagateNodeBeamChangesToDef`

**Contract** — clears the root module's nodes, beams, cinecams, all five wheel lists, shocks 1–3 and hydros, then regenerates them from the actor. Other sections are left as they were. Globals' dry and cargo mass are updated in place.

**Defaults compression** — the exporter walks elements in order and keeps a *current* node-defaults / beam-defaults / minimass / inertia-defaults / detacher-group record; whenever an element's values differ from the current record it starts a new record (which the serializer emits as a new `set_*` directive). Initial records are the built-ins (spring `DEFAULT_SPRING`, damp `DEFAULT_DAMP`, deform `BEAM_DEFORM`, break `BEAM_BREAK`, diameter `DEFAULT_BEAM_DIAMETER`, minimass 50 kg).

```text
nodes:   every node except tyre, rim and cinecam nodes; id = name or original number;
         position = spawn offset (tuned layout); load-weight override if ≥ 0; original options
beams:   only beams that came from 'beams'; options: support 's' (with extension break limit), rope 'r', invisible 'i';
         beam defaults from k, d, default deform, initial strength, diameter
         (for shock beams: the shock's sbd spring/damp/break)
cinecam: for each cinecam node, the 8 beams touching it → its 8 nodes; spring/damp from one of them; node mass
wheels:  by original keyword, with the parameters recorded at spawn; spring/damp read from the wheel's first beam;
         node defaults from its first tread node; beam defaults from its first beam
shocks:  every hydro beam bounded SHOCK1/2/3 (SHOCK1 + normal type is a wheel beam and is skipped);
         bounds, precompression and the type-specific parameters
hydros:  every entry of the hydro list; options: 'j' invisible (never 'i'), 's', 'e', 'r', 'a', 'n', and the combined 'v', 'y', 'h'
```

**Notes**
- Node references are written as the node's name when it has one, else its *original* number as text.
- Hydro option `i` also turns on steering input when it comes first (an old bug kept for compatibility), so the exporter always writes `j` for invisibility.
- Animators live in the same hydro list at runtime, so an exported animator comes back as a hydro. Combined-flag checks test "any of the bits", so `v`/`y`/`h` may be added more often than written originally. For `meshwheels2`/`flexbodywheels` the rim spring/damping are written into the *shared* current beam-defaults record, which may alter the defaults seen by earlier elements sharing it. All three are recorded behaviour of this proof-of-concept exporter, not format rules.
