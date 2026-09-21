# source/main/resources/tobj_fileformat/TObjFileFormat.cpp

> Reads and writes `.tobj` files; converts legacy road blocks into procedural roads.

**Needs** — [`TObjFileFormat.h`](TObjFileFormat.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`terrain/ProceduralRoad.h`](../../terrain/ProceduralRoad.h.md)
**Used by** — callers of [`TObjFileFormat.h`](TObjFileFormat.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Defines the `.tobj` grammar and its one piece of real logic: turning chains of old-style road-block objects into smooth procedural roads.

## State

```text
current line (raw and left-trimmed), line number, filename
default_rendering_distance : real = 0
rot_yxz                    : bool = false
preceding_comments         : text        # attached to the next entry
in_procedural_road         : bool
current procedural object, its start line
road2_last_pos, road2_last_rot, road2_num_blocks   # legacy road auto-import
```

## Line handling

**Contract** — lines are not trimmed except where noted. A line starting with `/` or `;` is a comment: its text (after any run of `/`) is collected into `preceding_comments`, which is attached to the next object/vehicle/road point and then cleared. Empty lines are skipped.

| Line (prefix) | Effect |
|---|---|
| `end` (exact) | stop reading |
| `collision-tris…` | ignored (obsolete) |
| `grid x, y, z` | grid position; grid enabled |
| `trees yaw_from, yaw_to, scale_from, scale_to, high_density, min_dist, max_dist, <mesh> <colormap> <densitymap> grid_spacing <collision_mesh>` | add tree layer |
| `grass range, sway_speed, sway_length, sway_distrib, density, min_x, min_y, max_x, max_y, grow_tech, min_h, max_h, <material> <colormap> <densitymap>` | add grass layer |
| `grass2` | as `grass` with `technique` (int) after `max_h`; technique outside 0..2 → 1 with a log line |
| `set_default_rendering_distance d` | applies to subsequent objects |
| `rot_yxz` | subsequent (and all, see Notes) rotations use Y-then-X-then-Z order |
| `begin_procedural_roads` | start a new procedural object (discarding any unfinished legacy road strip) |
| `end_procedural_roads` | flush the procedural object |
| `smoothing_num_splits n` (left-trimmed) | inside a procedural block |
| `collision_enabled bool` (left-trimmed) | inside a procedural block |
| inside a procedural block, anything else | a road point: `x, y, z, rx, ry, rz, width, border_width, border_height, <type>` |
| otherwise | an object line: `x, y, z, rx, ry, rz, <odef>[ <type>[ <instance>]]` (≥ 6 numbers else ignored) |

Road point types: `flat`, `left`, `right`, `both`, `bridge` (pillars), `bridge_no_pillars`, `monorail` (pillar type 2), `monorail2` (no pillars); anything else = automatic with pillars.

**Object lines** — the odef name is classified as a special object: `truck`, `load`, `machine`, `boat`, `truck2` → a pre-placed **vehicle** (the 8th column is the vehicle file); `grid`; road kinds `road`, `roadborderleft`, `roadborderright`, `roadborderboth`, `roadbridgenopillar`, `roadbridge` → **legacy road block**; anything else → a static object using that `.odef`. Objects without an instance name get `auto^<file>(line:<n>)`, so scripts can address every object. Each object gets the current default rendering distance and the collected comments.

## Rotations

```text
FUNCTION calc_rotation(deg, rot_yxz)
  IF rot_yxz: RETURN rot(Y, deg.y) * rot(X, deg.x) * rot(Z, deg.z)    # yaw global, then local pitch, roll
  ELSE:       RETURN rot(X, deg.x) * rot(Y, deg.y) * rot(Z, deg.z)
```

**Notes** — the `rot_yxz` flag is also stored on the document at the end, and the terrain object manager applies it when spawning static objects; since the flag is file-wide, a `rot_yxz` line anywhere effectively affects later lines here and all objects at spawn.

## Legacy road import

Old terrains built roads from individual road-block objects placed end to end. Consecutive blocks become one procedural road:

```text
FUNCTION process_road_object(obj)
  IF distance(obj.position, last_pos) > 20 m          # gap: start a new road
    IF road2_num_blocks > 0
      end_point = last_pos + calc_rotation(last_rot) * (10, 0, 0.9)   # extend past the last block
      import_point(end_point, last_rot, obj.special)
      flush_procedural_object()
    road2_num_blocks += 1
  import_point(obj.position, obj.rotation, obj.special)
  last_pos, last_rot = obj.position, obj.rotation

FUNCTION import_point(pos, rot, special)
  point = { position: pos, rotation: calc_rotation(rot), width: 8, border_width: 1.4, border_height: 0.2,
            type: FLAT if special == ROAD else AUTOMATIC,
            pillar_type: 0 if special == ROAD_BRIDGE_NO_PILLARS else 1,
            comments: preceding_comments }
  append to current procedural object (record start line if first)
```

At `Finalize`, an unfinished strip is closed the same way (end point at `last_pos + rot(last_rot) * (10, 0, 0.9)`, type ROAD). Flushing names the object `"<file> (lines <start> - <current>)"`.

## `WriteToStream(document, stream)`

**Contract** — writes sections in this order, each introduced by `//    ~~~~~~~~~~    <title> (<count>)    ~~~~~~~~~~` when non-empty: grid; trees; grass (always as `grass2`); vehicles (`x, y, z, rx, ry, rz, <kind> <file>` with original rotation); procedural roads (`begin_procedural_roads`, `smoothing_num_splits`, `collision_enabled`, one line per point `x, y, z, 0, yaw_deg, 0, width, bwidth, bheight, <type>` — only yaw is kept — then `end_procedural_roads`); static objects (`x, y, z, rx, ry, rz, <odef> <type> <instance>`, dropping `auto^` instance names). Comments are written back as `// text` lines before their entry.

**Notes** — the original's grid line is formatted without its arguments, which fails at runtime whenever a grid is present; a rebuild must write `grid x, y, z`.
