# source/main/physics/ApproxMath.h

> Fast approximate math and a tiny pseudo-random generator for the physics hot loop.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`gameplay/Engine.cpp`](../gameplay/Engine.cpp.md) · [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`gfx/GfxScene.cpp`](../gfx/GfxScene.cpp.md) · [`gfx/camera/CameraManager.cpp`](../gfx/camera/CameraManager.cpp.md) · [`ActorForcesEuler.cpp`](ActorForcesEuler.cpp.md) · [`ActorManager.cpp`](ActorManager.cpp.md) · [`ActorSpawner.cpp`](ActorSpawner.cpp.md) · [`physics/collision/Collisions.cpp`](collision/Collisions.cpp.md) · [`physics/flex/FlexAirfoil.cpp`](flex/FlexAirfoil.cpp.md) · [`physics/flex/FlexBody.cpp`](flex/FlexBody.cpp.md) · [`physics/flex/FlexMesh.cpp`](flex/FlexMesh.cpp.md) · [`physics/flex/FlexObj.cpp`](flex/FlexObj.cpp.md)
**Tier floor** — T2 (bit reinterpretation of floats)

## Purpose

The beam loop needs one inverse square root per beam per step; these bit-trick approximations were chosen for speed. A rebuild may use exact functions (modern hardware makes `1/sqrt` cheap) — the simulation does not depend on the approximation error, except that `fast_invSqrt` is accurate to ~0.2 % and anything coarser would alter beam forces.

## State

```text
mirand : int (32-bit, wraps) = 1     # global PRNG state; not thread-safe, only used for noise
```

## Random numbers

**Contract** — multiplicative LCG (`state *= 16807`, 32-bit wrap), then the low 23 bits are used as the mantissa of a float in [2, 4):
- `frand()` → [0, 1)
- `frand_02()` → [0, 2)
- `frand_11()` → [−1, 1)

Used only for aerodynamic turbulence noise and particle jitter; any fast uniform generator will do.

## Approximations

| Function | Method | Accuracy |
|---|---|---|
| `approx_exp(x)` | IEEE-754 exponent trick: `bits = 12102203·x + 1064652319`; 0 below −15, 1e38 above 88 | few % |
| `approx_pow2(x)` | `bits = 8388608·x + 1065353216` | few % |
| `approx_pow(x, y)` | scale `x`'s bit pattern relative to 1.0 by `y` | few % |
| `approx_sqrt(y)` | halve the exponent bits relative to 1.0 | ~6 % |
| `approx_invSqrt(y)` | magic constant `0x5f3759df − (bits >> 1)` | ~3.5 % |
| `fast_invSqrt(v)` | same + one Newton step `y·(1.5 − 0.5·v·y²)` | ~0.2 % |
| `fast_sqrt(x)` | `x · fast_invSqrt(x)` | ~0.2 % |
| `sign(x)` | −1, 0, +1 | exact |

Vector helpers `approx_normalise`, `fast_normalise`, `approx_length`, `fast_length` apply the above to a vector's squared length.
