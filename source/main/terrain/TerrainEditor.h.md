# source/main/terrain/TerrainEditor.h

> In-game terrain editor: select, move and rotate placed objects and predefined actors; write edits back to .tobj files.

**Needs** — [`Application.h`](../Application.h.md) · [`utils/memory/RefCountingObject.h`](../utils/memory/RefCountingObject.h.md)
**Used by** — [`scripting/bindings/TerrainAngelscript.cpp`](../scripting/bindings/TerrainAngelscript.cpp.md) · [`Terrain.h`](Terrain.h.md) · [`TerrainEditor.cpp`](TerrainEditor.cpp.md)
**Tier floor** — T2


## Purpose

A minimalist editor mode (toggled in simulation) and the editable object record exposed to scripts. Implementation: [`TerrainEditor.cpp`](TerrainEditor.cpp.md).

## State

```text
RECORD TerrainEditorObject                 # script-visible via getters/setters
  name (odef or truck file), instance_name, type ("-" = none), position, rotation (degrees),
  initial_position, initial_rotation, tobj_cache_id, tobj_comments
  static only: scene node, collision box ids, collision triangle ids, enable_collisions, script_handler
  predefined actor only: special_object_type (truck, truck2, load, machine, boat…), actor_instance_id

RECORD TerrainEditor
  object_tracking = true (the character follows the selection), rotation_axis = Y (0 X, 1 Y, 2 Z),
  last_object_name, selected_object_id = −1
```

## API

Editor: `UpdateInputEvents(dt)`, `WriteSeparateOutputFile`, `WriteEditsToTobjFiles`, `ClearSelectedObject`, `SetSelectedObjectByID`, `GetSelectedObjectID`, `FetchSelectedObject`. Object: `getPosition/getRotation/setPosition/setRotation`, name/instance/type getters, special type and actor id get/set.
