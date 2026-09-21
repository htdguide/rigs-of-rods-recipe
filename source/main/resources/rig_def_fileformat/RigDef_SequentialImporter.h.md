# source/main/resources/rig_def_fileformat/RigDef_SequentialImporter.h

> Converts legacy, order-dependent node numbering into the modern fixed node order.

**Needs** — [`RigDef_Prerequisites.h`](RigDef_Prerequisites.h.md) · [`RigDef_File.h`](RigDef_File.h.md)
**Used by** — [`RigDef_Parser.cpp`](RigDef_Parser.cpp.md) · [`RigDef_Parser.h`](RigDef_Parser.h.md) · [`RigDef_SequentialImporter.cpp`](RigDef_SequentialImporter.cpp.md)
**Tier floor** — T4

## Purpose

Old truck files address nodes by **index in a global node array** that was filled in file order — including nodes *generated* by `cinecam` and wheel lines. The modern spawner builds the node array in a fixed order instead:

1. `nodes` (numbered), 2. `nodes2` (named), 3. `cinecam` (1 per line), 4. `wheels` (rays × 2), 5. `wheels2` (rays × 4), 6. `meshwheels` (rays × 2), 7. `meshwheels2` (rays × 2), 8. `flexbodywheels` (rays × 4).

This class records every node as it is defined (in file order) and afterwards rewrites every numeric reference so that it points to the same physical node in the new order. Implementation: [`RigDef_SequentialImporter.cpp`](RigDef_SequentialImporter.cpp.md).

## State

```text
RECORD NodeMapEntry
  origin      : Keyword                 # which section created it
  detail      : RIM_A | RIM_B | TYRE_A | TYRE_B | UNDEFINED   # wheels only
  id          : NodeId                  # number or name
  sub_index   : int                     # index within its origin section's nodes

RECORD SequentialImporter
  enabled     : bool
  all_nodes   : list<NodeMapEntry>      # in file (legacy) order
  named_nodes : map<name, NodeMapEntry>
  counts per origin: numbered, named, cinecam, wheels, wheels2, meshwheels, meshwheels2, flexbodywheels
  statistics  : total resolved, resolved to self
  current keyword / module              # for messages
```

## Operations

`Init(enabled)`, `Disable()`, `IsEnabled()`, `AddNumberedNode(n)`, `AddNamedNode(name)`, `AddGeneratedNode(origin, detail)`, `GenerateNodesForWheel(origin, rays, has_rigidity)`, `Process(document)`, `GetNodeStatistics()`, `IterateAndPrintAllNodes()` — see `.cpp`.
