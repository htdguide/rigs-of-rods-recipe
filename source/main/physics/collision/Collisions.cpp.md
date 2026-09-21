# source/main/physics/collision/Collisions.cpp

> Ground model loading, the spatial hash, box/triangle/terrain contact for nodes, the character position fix-up, and the friction/fluid contact law.

**Needs** — [`Collisions.h`](Collisions.h.md) · [`Application.h`](../../Application.h.md) · [`ApproxMath.h`](../ApproxMath.h.md) · [`Actor.h`](../Actor.h.md) · [`ActorManager.h`](../ActorManager.h.md) · [`utils/ErrorUtils.h`](../../utils/ErrorUtils.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`gfx/GfxScene.h`](../../gfx/GfxScene.h.md) · [`gameplay/Landusemap.h`](../../gameplay/Landusemap.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`gfx/MovableText.h`](../../gfx/MovableText.h.md) · [`utils/PlatformUtils.h`](../../utils/PlatformUtils.h.md) · [`scripting/ScriptEngine.h`](../../scripting/ScriptEngine.h.md) · [`terrain/Terrain.h`](../../terrain/Terrain.h.md)
**Used by** — callers of [`Collisions.h`](Collisions.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

The static half of collision and, in `primitiveCollision`, the single contact law used by terrain, objects, meshes and actor-to-actor cabs.

## State

See [`Collisions.h`](Collisions.h.md).

## `primitiveCollision(node, v, mass, normal, dt, gm, penetration = 0) → force`

The contact law. `penetration` is how far the node is below the surface (0 for boxes/meshes, which only report contact).

```text
vn = v·normal; fn = node.forces·normal; F = 0
IF gm.solid_ground_level ≠ 0 AND penetration ≥ 0                        # inside a fluid layer (mud, snow, sand)
  μ = gm.flow_consistency_index · (|v|²)^((flow_behavior_index − 1)/2)   # power-law viscosity
  drag = −μ · node.surface_coef · v
  IF drag_anisotropy < 1 AND vn > 0                                      # moving out of the fluid: less drag
    k = min(|v|² / va², 1)
    drag += vn · μ · (1 − drag_anisotropy) · k · normal
  F += drag
  buoy = fluid_density · penetration · 9.807 · node.volume_coef
  IF flow_behavior_index < 1 AND vn ≥ 0 AND fn < 0 AND buoy > −fn: buoy = −fn   # pseudoplastic: only stops sinking
  F += buoy · normal
IF penetration ≥ gm.solid_ground_level                                    # touching solid ground
  R = −fn                                                                 # cancel the pushing force
  IF vn < 0: R −= (0.8·vn + 0.2·(solid_ground_level − penetration)/dt) · mass / dt   # stop the approach (80 %) + correct depth
  IF R > 0
    slip_force = node.forces − fn·normal; slip = v − vn·normal; s = |slip|; slip = unit(slip)
    G = R · gm.strength · node.friction_coef
    IF s < va AND G > 0 AND |slip_force| ≤ ms·G                          # static friction
      F += R·normal − ms·G·(1 − e^(−s/va))·slip − slip_force
    ELSE                                                                  # Stribeck sliding friction
      g = mc + (ms − mc)·e^(−(s/vs)^alpha)
      F += R·normal − (g + min(t2·s, 5))·G·slip
    node.avg_collision_slip = 0.995·avg + 0.005·s; node.last_collision_slip = s·slip
    node.last_collision_force = min(−R, 0)·normal
RETURN F
```

`e^x` and `x^y` use the fast approximations of [`ApproxMath.h`](../ApproxMath.h.md).

**Notes** — the reaction is computed from the node's *current accumulated force*, so this must run after all other forces of the step are summed for that node. It removes the normal component exactly and then adds the impulse needed to stop the node within one step; this is what makes a 2 kHz explicit integrator stable on hard ground.

## Ground models

**`loadGroundModelsConfigFile(file)`** — INI-style (`[section]`, `key = value`, separators tab/`:`/`=`). Section `general`/`config` holds `version` (must equal **3**, otherwise a fatal "Your ground configuration is too old" error and the program exits with code 124). Every other section is a ground model; defaults: alpha 2, strength 1, particle amount 20, min velocity 5, max 99999, velocity factor 0.7, fade −1, time delta 1, ttl 2. Keys: `adhesion velocity`, `static friction coefficient`, `sliding friction coefficient`, `hydrodynamic friction`, `stribeck velocity`, `alpha`, `strength`, `base`, `fx_type` (PARTICLE/HARD/DUSTY/CLUMPY), `fx_particle_name`, `fx_colour`, `fx_particle_*`, `fluid density`, `flow consistency index`, `flow behavior index`, `solid ground level`, `drag anisotropy`. After parsing, each model with a known `base` is replaced by a copy of the base and its own section is re-applied over it (single-level inheritance). The file is `ground_models.cfg` in the config directory, or a resource.

## Spatial hash

```text
cell(x, z) = (floor(x/2), floor(z/2))           # truncation toward zero
cell_id    = (cx << 16) + cz
bucket     = sbox_hash(cell_id) & (2^20 − 1)     # 4 bytes, each: h ^= SBOX[byte]; h *= 3   (fixed 256-entry table)
```

A box or triangle is added to every cell its AABB covers (cell coordinates clamped to 0..0x7FFF), and the bucket's height record is raised to the element's top. **Negative world coordinates clamp to cell 0**, so the static world is expected to lie in the positive quadrant (terrains are authored that way).

**Notes** — `nodeCollision` and `findPotentialEventBoxes` skip bucket entries whose `cell_id` differs (bucket collisions); `collisionCorrect` and `getSurfaceHeightBelow` do not, so they may also test elements from unrelated cells that share the bucket. Harmless (extra tests), but observable in performance.

## `addCollisionBox(rotating, virtual, pos, rot, lo, hi, selfrot, event, instance, forcecam, campos, scale, dir, filter, script_handler, reverb)`

```text
relo, rehi = lo·scale, hi·scale; selfcenter = (relo + rehi)/2; center = pos
rot quaternion = X(rot.x)·Y(rot.y)·Z(rot.z) (degrees); same composition for self-rotation and event direction
refined = any |rot component| ≥ 0.0001
IF event name given: allocate the next event source (instance, box name, handler, box index, direction, enabled)
IF refined OR self-rotated: world AABB = bounds of the 8 corners after self-rotation about selfcenter, then global rotation, + pos
ELSE AABB = pos + relo .. pos + rehi
register in hash; grow world_aab; RETURN box index
```

`removeCollisionBox/Tri` only disable (and disable the event source); the hash is not updated.

## `addCollisionTri(p1, p2, p3, gm)`

Stores the triangle with basis `bx = p2 − p1`, `by = p3 − p1`, `bz = unit(bx × by)`; `reverse = [bx by bz]`, `forward = reverse⁻¹`; AABB padded by 0.1 m; hashed. `addCollisionMesh` loads a mesh, transforms its vertices by scale → rotation → translation, adds one triangle per index triple (default ground model concrete) and records a mesh entry; `registerCollisionMesh` records an entry for triangles added elsewhere.

## `nodeCollision(node, dt) → contacted`

```text
bucket for node's cell; IF node.y > bucket height: RETURN false
FOR EACH element in bucket with this cell id
  box (enabled, node inside world AABB):
    p = node position in box-local frame (undo global rotation, then self-rotation about selfcenter)
    IF inside relo..rehi (or the box is axis-aligned)
      IF box forces camera and none forced yet: force camera to box.campos
      IF NOT virtual
        normal = the face nearest to p among −z, +z, −x, +x, −y, +y (ties: first in that order), rotated back to world
        node.forces += primitive_collision(node, v, m, normal, dt, default_gm)     # boxes are "concrete"
  tri (enabled, node inside padded AABB):
    q = forward · (node − a)
    IF q.x ≥ 0, q.y ≥ 0, q.x + q.y ≤ 1, −0.1 < q.z < 0: keep the one with smallest −q.z
IF a triangle was kept: normal = reverse · (0,0,1); node.forces += primitive_collision(…, tri.gm)
```

Triangles only collide from the *front* (normal side), within a 10 cm layer behind the surface. Every node gets at most one triangle contact but may get several box contacts.

## `groundCollision(node, dt)`

Returns whether the node touched the ground.


Terrain height h at (x, z); if above the node: ground model from the land-use map (else gravel), normal from the terrain, force = `primitive_collision(node, v, m, normal, dt, gm, penetration = h − y)`.

## `collisionCorrect(position, callbacks) → contacted`

For the walking character: moves the point out of boxes (to the nearest face, in the box frame) and onto the nearest triangle under it; fires event-box script callbacks (once per box until the character leaves all boxes — the "last called" cache), honours camera-forcing boxes.

## Other queries

- `getSurfaceHeightBelow(x, z, h)` — max of terrain height and the tops of non-virtual boxes and triangles under (x, z) that are below h (vertical ray from the bucket height). `getSurfaceHeight` = no ceiling.
- `intersectsTris(ray)` — steps along the ray at cell spacing, tests triangles in each new bucket, returns the first hit with ray parameter < 1.
- `intersectsTerrain(ray)` — samples every 0.1 m along the ray's length; first sample below terrain returns its fraction of the length.
- `isInside(pos, box, border)` — AABB test grown by `border`, then exact test in the box frame.
- `findPotentialEventBoxes(actor, out)` — enabled event boxes in the cells covered by the actor's event bounding box, filtered by `permitEvent`: ALL; AVATAR (character only); TRUCK / TRUCK_WHEELS (actor type truck; the wheel check happens elsewhere); AIRPLANE; BOAT.
- `getPosition/getDirection/getBox(instance, box)` — linear search of event sources.
- `envokeScriptCallback(box, node?)` — only on the main thread; skipped for disabled sources.
- Debug visualisation: a coloured quad per occupied cell, colour by bucket fill relative to 126.
