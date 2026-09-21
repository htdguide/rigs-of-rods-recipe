# source/main/physics/air/Airfoil.cpp

> Parses .afl coefficient tables and evaluates coefficients with a control-surface correction.

**Needs** — [`Airfoil.h`](Airfoil.h.md) · [`Application.h`](../../Application.h.md)
**Used by** — callers of [`Airfoil.h`](Airfoil.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Turns a published airfoil table into a dense lookup, then applies a simple flap/aileron model.

## State

See [`Airfoil.h`](Airfoil.h.md).

## `Airfoil(file)`

```text
zero all tables
find the file in any resource group (else log "Airfoil error: could not load airfoil <file>" and keep zeros)
skip lines until one starting with "alpha"
FOR EACH following line "A.B cl cd cm"          # angle written as integer part and one tenth digit
  B = −B while still in the negative range (until the row 0.0 is seen)
  index = A·10 + B + 1800
  store coefficients; IF index == 3600: stop after this row
  linearly interpolate any skipped indices since the previous row
```

**Notes** — the angle is parsed as two integers around the dot, so exactly one decimal digit is assumed; negative angles rely on the sign of the integer part plus the "still negative" flag. Keep this parser for compatibility with the shipped .afl files.

## `getparams(a, cratio, cdef)`

```text
va = a wrapped into [−180, 180]; i = floor((va + 180)·10)
dva = va + 1.15·(1 − cratio)·cdef wrapped; di = floor((dva + 180)·10)      # drag curve shifted by deflection
s = sign(cdef)
cl = CL[i] − 0.66·s·(1 − cratio)·sqrt|cdef|
cd = CD[di] + 0.00015·(1 − cratio)·cdef²
cm = CM[i] + 0.20·s·(1 − cratio)·sqrt|cdef|
```

`cratio` is the fraction of chord ahead of the hinge (1 = no control surface); `cdef` the surface deflection in degrees.
