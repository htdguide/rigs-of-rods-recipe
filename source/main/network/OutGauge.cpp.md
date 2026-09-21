# source/main/network/OutGauge.cpp

> Builds and sends one OutGauge packet per configured interval.

**Needs** — [`OutGauge.h`](OutGauge.h.md) · [`Application.h`](../Application.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`gui/DashBoardManager.h`](../gui/DashBoardManager.h.md) · [`gameplay/Engine.h`](../gameplay/Engine.h.md) · [`RoRVersion.h`](../../../source/version_info/RoRVersion.h.md) · [Seam: UDP datagrams](../../../SYSTEM-REQUIREMENTS.md#seam-udp-datagrams)
**Used by** — callers of [`OutGauge.h`](OutGauge.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`OutGauge.h`](OutGauge.h.md).

## State

See header.

## `Connect`

Resolve `io_outgauge_ip`, create a UDP socket and connect it to that address and `io_outgauge_port` (fixes the destination for later sends). On success, working = true.

**Notes** — implemented only on Windows; elsewhere Connect does nothing and Update returns false. Nothing Windows-specific is needed; a rebuild supports every platform.

## `Update(dt, vehicle)`

```text
IF not working: RETURN false
timer += dt; IF timer < 0.1 × io_outgauge_delay: RETURN true      # delay is in tenths of a second
timer = 0
pack = zeros; Time = ms clock; ID = io_outgauge_id; Flags = KM; Car = "RoR"
no vehicle → Display2 "not in vehicle"; no engine → "no engine"
else:
  TURBO flag if it has a turbo; Gear = max(0, gear + 1)            # one reverse gear only
  Speed = |wheel speed|; RPM; Turbo = turbo psi × 0.0689475729 (bar)
  temps, fuel, oil = 0 (not simulated)
  available lights: HANDBRAKE, BATTERY, SIGNAL_L/R/ANY, TC and ABS unless hidden on the dashboard
  lit: parking brake → HANDBRAKE; headlights → FULLBEAM; ignition without running → BATTERY;
       turn signals / hazard → SIGNAL_L / SIGNAL_R / SIGNAL_ANY; TC mode → TC; ABS mode → ABS
  Throttle = accelerator; Brake; Clutch = 1 − engine clutch
  Display1 = first 15 chars of vehicle name, Display2 = next 15
send pack; RETURN true
```

## `Close`

Closes the socket if open.
