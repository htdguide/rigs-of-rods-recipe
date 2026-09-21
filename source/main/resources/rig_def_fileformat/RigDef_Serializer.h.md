# source/main/resources/rig_def_fileformat/RigDef_Serializer.h

> Writes a truck document back to truck-file text.

**Needs** — [`RigDef_File.h`](RigDef_File.h.md)
**Used by** — [`physics/ActorManager.cpp`](../../physics/ActorManager.cpp.md) · [`RigDef_Serializer.cpp`](RigDef_Serializer.cpp.md)
**Tier floor** — T4

## Purpose

Used to save modified vehicles (tuning/project export from [`ActorManager`](../../physics/ActorManager.cpp.md)). Implementation: [`RigDef_Serializer.cpp`](RigDef_Serializer.cpp.md).

## State

```text
RECORD Serializer
  document  : Document
  output    : text buffer
  widths    : node_id 5, float 10, bool 5, command_key 2, inertia_function 10   # column padding
  indents   : data-line indent "", set-defaults indent ""
  current_beam_defaults, current_node_defaults, current_default_minimass   # last written presets
```

## `Serialize()` · `GetOutput()`

See `.cpp` twin.
