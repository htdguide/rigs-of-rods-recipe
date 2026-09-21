# source/main/physics/collision/DynamicCollisions.cpp

> Node-vs-cab-triangle contact: adaptive skipping, barycentric inside test, backface heuristic, and force distribution to the triangle vertices.

**Needs** — [`DynamicCollisions.h`](DynamicCollisions.h.md) · [`Application.h`](../../Application.h.md) · [`Actor.h`](../Actor.h.md) · [`SimData.h`](../SimData.h.md) · [`CartesianToTriangleTransform.h`](CartesianToTriangleTransform.h.md) · [`Collisions.h`](Collisions.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`PointColDetector.h`](PointColDetector.h.md) · [`Triangle.h`](Triangle.h.md)
**Used by** — callers of [`DynamicCollisions.h`](DynamicCollisions.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Handles collisions between *moving* bodies (actor cabs), as opposed to [`Collisions`](Collisions.cpp.md), which handles the static world. Called from the per-step work (inter-actor pass in parallel per actor; intra-actor pass inside each actor's step).

## State

Per collision cab: a skip counter pair `{rate, distance}` owned by the actor (intra and inter separately).

## `ResolveCollisionForces(depth, hit, na, nb, no, α, β, γ, normal, dt, remote, gm)`

```text
v_rel   = hit.velocity - (α·na.v + β·nb.v + γ·no.v)
m_tri   = α·na.m + β·nb.m + γ·no.m
m       = remote ? hit.m : hit.m·m_tri / (hit.m + m_tri)        # reduced mass; remote actors are not simulated here
F       = primitive_collision(hit, v_rel, m, normal, dt, gm, depth)   # the ground contact law, see Collisions
hit.forces += F; na −= α·F; nb −= β·F; no −= γ·F                # equal and opposite, split barycentrically
```

## `ResolveInterActorCollisions`

```text
FOR EACH collision cab i
  IF rate[i] > 0: distance[i]++; rate[i]--; CONTINUE              # skip this step
  rate[i] = min(distance[i], 12); distance[i] = 0
  (no, na, nb) = the cab's three nodes
  detector.query(no, na, nb, enlarge = collision range)
  IF no hits: rate[i]++ ; CONTINUE                                # back off further next time
  tri = Triangle(na, nb, no); T = CartesianToTriangleTransform(tri)
  FOR EACH hit actor, FOR EACH of its hit nodes h
    (α, β, γ, d) = T(h.pos)
    IF α, β, γ ≥ 0 AND |d| ≤ range                               # inside the triangle prism
      rate[i] = 0
      n = tri.normal
      IF backface(d, n, no, neighbours of h): n = −n; d = −d
      resolve_collision_forces(range − d, h, na, nb, no, α, β, γ, n, dt, remote = h's actor is networked, submesh gm)
      h.last_collision_gm = submesh gm; mark h, na, nb, no as having mesh contact
```

**Backface heuristic** — score = 3·sign(d); if the hit node has more than 3 neighbours, add sign(n·(neighbour − no)) for each; negative score ⇒ the contact is on the back side. A node pushed partly through a panel is thus pushed back the way it came rather than through.

**Adaptive skipping** — a cab that finds nothing nearby checks progressively less often (skip interval grows by one each empty check, capped near 13 steps); any hit, or two actors closing faster than 4 m/s, resets it to every step.

## `ResolveIntraActorCollisions`

As above with the actor's own nodes, except: tyre nodes and the triangle's own vertices are ignored; the backface test is simply `d < 0`; the remote flag is never set; after a collision the counter is set to −20000 (checked every step for about 10 s); the skip interval is restored from `distance` only when it was positive.
