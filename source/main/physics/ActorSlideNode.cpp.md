# source/main/physics/ActorSlideNode.cpp

> Actor-level slide-node operations: lock/unlock to the nearest rail on any permitted actor, and per-step updates.

**Needs** — [`SlideNode.h`](SlideNode.h.md) · [`Actor.h`](Actor.h.md) · [`GameContext.h`](../GameContext.h.md)
**Used by** — callers of [`Actor.h`](Actor.h.md) — it implements the actor's slide-node methods
**Tier floor** — T2

## Purpose

Methods of [`Actor`](Actor.h.md) that operate over all of its slide nodes; split out of the actor file by topic.

## State

Uses the actor's `slidenodes`, `railgroups`, `slidenodes_locked`.

## `toggleSlideNodeLock`

**Contract** — for each slide node that may attach anywhere (self or foreign): if currently locked, detach it; otherwise search every actor it is allowed to attach to (own actor only if `attach_self`, other actors only if `attach_foreign`) for the rail whose closest segment is nearest and within the node's `attach_distance`, and attach to it (or detach if none). Then flips `slidenodes_locked`.

## `GetClosestRailOnActor(actor, node)`

**Contract** — over the actor's rail groups (skipping empty entries): the group whose closest segment is within attach distance and nearest; returns (group or none, distance).

## Per-step helpers

- `updateSlideNodeForces(dt)` — for each slide node: update position, then forces (called from the physics step).
- `updateSlideNodePositions` — positions only. `resetSlideNodePositions` — recompute nearest segments (after moves/resets). `resetSlideNodes` — back to initial rails, un-break.

**Notes** — lock toggling scans every rail of every actor: quadratic, but user-triggered, not per step. Rail-to-slide-node links to *other* actors are one of the ways actors become "linked" for the purposes of pausing and command forwarding.
