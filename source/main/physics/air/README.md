# source/main/physics/air

> Aircraft parts that add forces to nodes: engines, airbrakes, and the airfoil tables everything aerodynamic reads.

Wings themselves are flexing meshes and live in [`../flex/FlexAirfoil`](../flex/FlexAirfoil.h.md); they use [`Airfoil`](Airfoil.h.md) from here. All aero parts share the ISA troposphere density model `ρ = 101325·(1 − 0.0065·h/288.15)^5.24947 · 1.20896e-5`.

## Reading order

1. [`Airfoil.h`](Airfoil.h.md) → [`Airfoil.cpp`](Airfoil.cpp.md)
2. [`AeroEngine.h`](AeroEngine.h.md)
3. [`TurboJet.h`](TurboJet.h.md) → [`TurboJet.cpp`](TurboJet.cpp.md)
4. [`TurboProp.h`](TurboProp.h.md) → [`TurboProp.cpp`](TurboProp.cpp.md)
5. [`AirBrake.h`](AirBrake.h.md) → [`AirBrake.cpp`](AirBrake.cpp.md)
