# source/main/physics/water

> Water as physics: the wave surface, hull buoyancy/drag, and boat propellers.

The rendered water is a pluggable seam elsewhere; this chapter only answers "where is the surface and how does it move". Waves are disabled in multiplayer so all peers see flat, identical water.

## Reading order

1. [`Wavefield.h`](Wavefield.h.md) → [`Wavefield.cpp`](Wavefield.cpp.md)
2. [`Buoyance.h`](Buoyance.h.md) → [`Buoyance.cpp`](Buoyance.cpp.md)
3. [`ScrewProp.h`](ScrewProp.h.md) → [`ScrewProp.cpp`](ScrewProp.cpp.md)
