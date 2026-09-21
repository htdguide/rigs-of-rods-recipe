# source/main/resources/rig_def_fileformat/RigDef_Node.h

> How the truck format names nodes, refers to them, and expresses node ranges.

**Needs** — [`utils/BitFlags.h`](../../utils/BitFlags.h.md) · [`RigDef_Prerequisites.h`](RigDef_Prerequisites.h.md)
**Used by** — [`RigDef_File.h`](RigDef_File.h.md) · [`RigDef_Node.cpp`](RigDef_Node.cpp.md)
**Tier floor** — T4

## Purpose

Nodes are the point masses of a vehicle. In the file they are declared either **numbered** (`nodes` section: `0, x, y, z`) or **named** (`nodes2`: `wheel_fl, x, y, z`), and every other section refers to them by the text of that id. Because legacy files (see [`RigDef_SequentialImporter`](RigDef_SequentialImporter.h.md)) resolved references *in file order* and could mix numbers with generated nodes, a reference carries two interpretations until the file is fully read. Implementation of the helpers: [`RigDef_Node.cpp`](RigDef_Node.cpp.md).

## State

```text
RECORD NodeId                          # a node's own identity
  text   : text                        # always set; for numbered nodes the decimal number
  number : int (unsigned)              # meaningful only if numbered
  kind   : VALID? + (NUMBERED | NAMED)

RECORD NodeRef                         # a reference from some other section
  text        : text                   # as written in the file
  as_number   : int (unsigned)         # parsed number (absolute value) for legacy import
  line        : int                    # source line, for messages
  # two interpretation states kept side by side:
  import_state  : VALID?, MUST_CHECK_NAMED_FIRST?, RESOLVED_NAMED?, RESOLVED_NUMBERED?
  regular_state : VALID?, NAMED?, NUMBERED?
  # equality compares `text` only

RECORD NodeRange
  start, end : NodeRef                 # single node when start == end

RECORD Node                            # one line of 'nodes' / 'nodes2'
  id                  : NodeId
  position            : (x, y, z) real  # metres, vehicle space
  options             : BitMask         # see below
  load_weight_override: real, present?  # only with option 'l'
  node_defaults       : shared NodeDefaults   # the 'set_node_defaults' in force at this line
  default_minimass    : shared DefaultMinimass or none
  beam_defaults       : shared BeamDefaults    # needed when this node becomes a hook
  detacher_group      : int = 0
```

"Import state" is the legacy (numbered, order-dependent) interpretation; "regular state" is the modern named interpretation. The importer decides at the end which one applies (see [`RigDef_SequentialImporter.h`](RigDef_SequentialImporter.h.md)).

## Node options

One letter each, any order, in the 5th column of a node line:

| Letter | Meaning |
|---|---|
| `n` | no-op placeholder |
| `m` | cannot be grabbed with the mouse |
| `f` | no sparks when scraping |
| `x` | exhaust point (origin of exhaust smoke) |
| `y` | exhaust direction reference |
| `c` | no ground contact (does not collide with terrain) |
| `h` | hook point (a hook is created on this node) |
| `e` | terrain-editor point (legacy) |
| `b` | extra buoyancy |
| `p` | no particles |
| `L` | log this node's state (debug) |
| `l` | has an explicit load weight (value in 6th column) |
