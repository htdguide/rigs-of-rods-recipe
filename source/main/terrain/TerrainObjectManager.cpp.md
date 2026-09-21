# source/main/terrain/TerrainObjectManager.cpp

> Loading .tobj/.odef content into scene objects, collisions, events, lights, animations and map icons; predefined actors.

**Needs** — [`TerrainObjectManager.h`](TerrainObjectManager.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`Application.h`](../Application.h.md) · [`gameplay/AutoPilot.h`](../gameplay/AutoPilot.h.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`physics/collision/Collisions.h`](../physics/collision/Collisions.h.md) · [`system/Console.h`](../system/Console.h.md) · [`utils/ErrorUtils.h`](../utils/ErrorUtils.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`gui/GUIManager.h`](../gui/GUIManager.h.md) · [`gui/panels/GUI_LoadingWindow.h`](../gui/panels/GUI_LoadingWindow.h.md) · [`utils/MeshObject.h`](../utils/MeshObject.h.md) · [`resources/odef_fileformat/ODefFileFormat.h`](../resources/odef_fileformat/ODefFileFormat.h.md) · [`utils/PlatformUtils.h`](../utils/PlatformUtils.h.md) · [`ProceduralRoad.h`](ProceduralRoad.h.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md) · [`TerrainGeometryManager.h`](TerrainGeometryManager.h.md) · [`Terrain.h`](Terrain.h.md) · [`resources/terrn2_fileformat/Terrn2FileFormat.h`](../resources/terrn2_fileformat/Terrn2FileFormat.h.md) · [`resources/tobj_fileformat/TObjFileFormat.h`](../resources/tobj_fileformat/TObjFileFormat.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [`utils/WriteTextToTexture.h`](../utils/WriteTextToTexture.h.md) · [`gfx/particle/ExtinguishableFireAffector.h`](../gfx/particle/ExtinguishableFireAffector.h.md)
**Used by** — callers of [`TerrainObjectManager.h`](TerrainObjectManager.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`TerrainObjectManager.h`](TerrainObjectManager.h.md).

## State

See header.

## `LoadTObjFile(name)`

Parse with the [TObj parser](../resources/tobj_fileformat/TObjFileFormat.cpp.md) from the terrain's group (errors to console, file skipped) and cache the document; then:

- `grid` → a 50 m reference grid of 1 m lines (red/blue centre lines) at the given position;
- `trees`, `grass` → vegetation (only if vegetation is enabled, and only with the paging seam), each wrapped so a bad line is reported and skipped;
- procedural roads → added to the procedural manager;
- vehicles → recorded as *editor objects* with a special type (spawned later);
- object lines → `LoadTerrainObject`, remembering which tobj they came from and their preceding comments.

### Trees

Density map (bilinear) and colour map; batch-page LOD from `min_distance × detail factor` (≥ 10), impostor LOD from `max_distance × factor` (≥ 10) when farther than the batch fade; page size 50. Placement: grid mode (`grid_spacing > 0`) puts one tree at each cell centre where density ≥ 0.8; random mode uses cells of 10 m (or −spacing) and places `high_density × density × detail factor` trees at random (negative high density = random up to that). Random yaw and scale in the given ranges. A collision mesh, when named, is added per tree at the terrain height with the tree's yaw (scaled ×0.1 in grid mode).

### Grass

One grass layer: sway speed/length/distribution, density × detail factor, size range, height range, technique (values > 10 select the same technique with blending), optional colour/density maps (`none` to skip), fade technique 0 grow / 1 alpha-grow / 2 alpha, range × detail factor, rendered before the main queue.

## `LoadTerrainObject(name, pos, rot, instance, type, …)`

```text
IF type == "grid": place the object on a 10×10 grid with 50 m spacing (instance = object name); RETURN
odef = cached or parsed "<name>.odef" from any group (missing → console error in simulation, false)
scene node: mesh (unless "none") with shadows per ODEF and rendering distance; scale; position;
  orientation = X(rot.x)·Y(rot.y)·Z(rot.z) degrees then pitch −90° (object files are Z-up), undone (+90°) for "standard" mode ODEFs
record an editor object (initial position/rotation, collisions flag, script handler, source tobj)
uniquify materials per instance when requested: clone "<material>/<instance>"
localizers → recorded at pos with the object rotation
sounds → positional terrain sound instances started immediately
ground model files → loaded into collisions
race objects (instance starts with "checkpoint" or "race"): map icon "checkpoint", or "racestart" for "race…" or a 4-part "…|id|0|…" name; race id = second '|' field
other typed objects (type not "" / "-"): map icon "icon_<type>.dds"; station/hotel/village/observatory/farm/ship/sign get caption "<instance> <type>"
collision boxes (if collisions enabled; race boxes only if sim_races_enabled): invalid (min > max) skipped with warning;
  added with the object's position/rotation, ODEF rotation, event name, instance, reverb, forced camera, scale, direction, filter, handler
collision meshes → collision triangles with the node orientation and the named ground model
particle systems (unique names by appending '_'), with the instance name passed to extinguishable-fire affectors
material override; RTSS-generated material; animations (speed random in [min, max]; missing ones logged)
texture prints: clone the base texture, draw text (with "{{argument1}}" → instance name, '_' → space) in the given box, font, colour, size, DPI
spot and point lights with a flare billboard sized clamp(range/10, 0.2, 2)
RETURN true
```

## Other

- `destroyObject(instance)` — predefined actor → queue its deletion; static object → destroy its scene objects and disable its collision boxes/triangles; deselect in the editor; remove from the list.
- `LoadPredefinedActors` (not in multiplayer) — spawn each special object except boats on dry terrains. `SpawnSinglePredefinedActor` — reserve an instance id once (reuse and respawn only if the actor no longer exists); spawn request with origin TERRN_DEF at the object position, rotation from the tobj's `rot_yxz` flag (unknown tobj → warning, false), free position for `truck2`, machine flag for `machine`.
- `LoadTelepoints` — one map icon per terrn2 telepoint.
- `UpdateTerrainObjects(dt)` — update vegetation paging, advance animations by dt × speed, adjust particle time factors.
- `LoadTerrainScript(file)` — load a script unit under its own scene grouping node.
