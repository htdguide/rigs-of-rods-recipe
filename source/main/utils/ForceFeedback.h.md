# source/main/utils/ForceFeedback.h

> Drives a steering wheel's constant-force effect from vehicle steering stress.

**Needs** — [Seam: Windowing and input devices](../../../SYSTEM-REQUIREMENTS.md#seam-windowing-and-input-devices)
**Used by** — [`AppContext.h`](../AppContext.h.md) · [`ForceFeedback.cpp`](ForceFeedback.cpp.md) · [`InputEngine.h`](InputEngine.h.md)
**Tier floor** — T2

## Purpose

Makes a force-feedback wheel resist steering in proportion to the load on the vehicle's steering hydraulics, plus speed-dependent self-centering. Implementation: [`ForceFeedback.cpp`](ForceFeedback.cpp.md).

## State

```text
RECORD ForceFeedback
  device  : optional<FF device>
  effect  : optional<constant-force effect>   # created lazily on first use
  enabled : bool                               # false when not driving
```

## `Setup()` · `SetEnabled(bool)` · `Update()`

**Contract** — see the `.cpp` twin.
