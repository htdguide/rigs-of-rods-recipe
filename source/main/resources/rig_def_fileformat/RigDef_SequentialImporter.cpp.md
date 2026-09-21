# source/main/resources/rig_def_fileformat/RigDef_SequentialImporter.cpp

> Node registration during parsing and reference rewriting afterwards.

**Needs** — [`RigDef_SequentialImporter.h`](RigDef_SequentialImporter.h.md) · [`RigDef_Parser.h`](RigDef_Parser.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`Application.h`](../../Application.h.md) · [`system/Console.h`](../../system/Console.h.md)
**Used by** — callers of [`RigDef_SequentialImporter.h`](RigDef_SequentialImporter.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Makes legacy numbered references land on the right node. Messages go to the console (area ACTOR) with the module and keyword appended.

## State

See header twin.

## Registration (during parsing)

```text
FUNCTION add_numbered_node(n)
  IF n != size(all_nodes)                        # numbered nodes must be consecutive from 0
    FATAL "Lost sync in node numbers, got [n], expected [size]. Ignoring node."; RETURN false
  append (NODES, id n, sub_index = numbered_count++)

FUNCTION add_named_node(name)
  IF name already registered: FATAL "Duplicate node name [name]. Ignoring node."; RETURN false
  entry = (NODES2, id name, sub_index = named_count++); register by name; append

FUNCTION add_generated_node(origin, detail)
  append (origin, id = size(all_nodes), sub_index = <origin>_count++, detail)

FUNCTION generate_nodes_for_wheel(origin, rays, _)
  IF origin IN {wheels2, flexbodywheels}
    FOR i IN 0..2*rays-1: add_generated_node(origin, RIM_A if i even else RIM_B)
  FOR i IN 0..2*rays-1:   add_generated_node(origin, TYRE_A if i even else TYRE_B)
```

Note that `wheels` and meshwheels generate `2 × rays` nodes; `wheels2` and `flexbodywheels` generate `4 × rays` (rim ring then tyre ring), alternating between the two axle sides.

## New position of a node

```text
FUNCTION offset(origin)      # cumulative counts of all sections that come earlier in the new order
  order = [NODES, NODES2, CINECAM, WHEELS, WHEELS2, MESHWHEELS, MESHWHEELS2, FLEXBODYWHEELS]
  RETURN sum of counts of the sections before `origin` in `order`

new_index(entry) = offset(entry.origin) + entry.sub_index
```

## `Process(document)`

**Contract** — for the root module and then every user module, rewrite every node reference in every section that has them (airbrakes, axles, beams, cameras, camera rails, cinecams, collision boxes, commands, contacters, exhausts, extcamera, fixes, flares2, flexbodies (+forset), flexbody wheels, fusedrag, hooks, hydros, mesh wheels (both), particles, props, rail groups, ropables, ropes, rotators (both), screwprops, shocks (all three), slidenodes (+rail ranges), soundsources (both), submesh texcoords and cab triangles, ties, triggers, turbojets, turboprops, wheels (both), videocameras, wings). Optionally logs statistics (`diag_rig_log_node_stats`) and a full node listing (`diag_rig_log_node_import`).

Sections **not** rewritten (e.g. `flares3`, `animators`, `lockgroups`, `add_animation`) keep their references as written.

```text
FUNCTION resolve(ref)
  IF ref not import-valid: RETURN ref unchanged
  IF ref.must_check_named_first AND ref.text is a registered name
    RETURN named reference to that name
  IF ref.number >= size(all_nodes)
    ERROR "Cannot resolve …"; RETURN numbered reference to node 0   # exactly what 0.38 did
  entry = all_nodes[ref.number]
  IF entry is named:    RETURN named reference to entry's name
  IF entry is numbered: RETURN numbered reference to new_index(entry)
  ERROR; RETURN invalid reference
```

Generated nodes are recorded with numbered ids, so references to them become numbered references into the new layout.

## Range expansion

- **Rail groups and slidenode rails**: a single reference is resolved as above; a range `a–b` expands to indices `a … b−1` (**end exclusive** — an original quirk) each resolved by index; ranges containing invalid or already-named ends are dropped with an error.
- **Flexbody `forset`**: every item is treated as a numeric index; ranges expand `a … b` **inclusive**; entries that fail to resolve are silently dropped (with a warning), because the legacy flexbody code ignored unknown nodes.

## Diagnostics

`GetNodeStatistics` prints totals per origin and each origin's start/end index after conversion; `IterateAndPrintAllNodes` lists every node in legacy order with origin, sub-index and wheel detail.
