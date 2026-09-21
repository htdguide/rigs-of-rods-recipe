# source/main/physics/SlideNode.h

> Rails (chains of beams) and slide nodes (nodes constrained to slide along a rail by a spring).

**Needs** — [`ForwardDeclarations.h`](../ForwardDeclarations.h.md)
**Used by** — [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`Actor.cpp`](Actor.cpp.md) · [`ActorSlideNode.cpp`](ActorSlideNode.cpp.md) · [`ActorSpawner.cpp`](ActorSpawner.cpp.md) · [`SlideNode.cpp`](SlideNode.cpp.md)
**Tier floor** — T2

## Purpose

Declares the three records of the slide-node feature: a rail segment, a rail group, and a slide node. Behaviour is in [`SlideNode.cpp`](SlideNode.cpp.md); actor-level operations (lock toggle, per-step update) are in [`ActorSlideNode.cpp`](ActorSlideNode.cpp.md).

## State

```text
RECORD RailSegment
  beam : beam                      # an existing beam of the actor
  prev, next : RailSegment?        # neighbours in the chain; a looped rail links last ↔ first

RECORD RailGroup
  segments : list<RailSegment>     # in chain order
  id       : int = -1              # 'railgroups' id, used to match slide nodes at spawn

RECORD SlideNode
  node                 : node      # the sliding node
  beam                 : beam?     # segment beam currently slid on (none = detached)
  initial_rail, current_rail : RailGroup?
  current_segment      : RailSegment?
  forces_ratio         : 0..1      # position of the ideal point along the beam (0 = p1, 1 = p2)
  ideal_position       : Vec3      # nearest point on the beam (world)
  initial_threshold, current_threshold : m   # free play before the spring engages
  spring_rate          : N/m = 9 000 000
  break_force          : N = +∞    # never breaks unless set
  attach_rate          : m/s = 1   # how fast current_threshold shrinks back after attaching
  attach_distance      : m = 0.1   # max reach when looking for a rail
  attach_self, attach_foreign, broken : bool
```

All setters store absolute values. `SetDefaultRail` sets initial and current rail and recomputes the position; `ResetSlideNode` returns to the initial rail and clears `broken`; `AttachToRail(rail?)` switches rail (none = detach), recomputes, and sets `current_threshold` to the current distance to the beam, so the node is pulled in gradually rather than snapped.
