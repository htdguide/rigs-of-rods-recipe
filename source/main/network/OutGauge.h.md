# source/main/network/OutGauge.h

> Streams the player vehicle to external dashboards using the Live for Speed OutGauge UDP packet.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`Application.cpp`](../Application.cpp.md) · [`main.cpp`](../main.cpp.md) · [`OutGauge.cpp`](OutGauge.cpp.md)
**Tier floor** — T2


## Purpose

Telemetry for third-party gauge apps and hardware ([Seam: UDP datagrams](../../../SYSTEM-REQUIREMENTS.md#seam-udp-datagrams)). Implementation: [`OutGauge.cpp`](OutGauge.cpp.md).

## State

```text
RECORD OutGauge: working bool, timer seconds, socket
OutGaugePack (packed, little-endian, 96 bytes — layout owned by Live for Speed)
  Time u32 ms, Car[4], Flags u16, Gear u8 (0 R, 1 N, 2 first…), PLID u8,
  Speed f32 m/s, RPM f32, Turbo f32 bar, EngTemp f32 °C, Fuel f32 0–1, OilPressure f32 bar, OilTemp f32 °C,
  DashLights u32 (available), ShowLights u32 (lit), Throttle f32, Brake f32, Clutch f32,
  Display1[16], Display2[16], ID i32
Flags: SHIFT 1, CTRL 2, TURBO 8192, KM 16384, BAR 32768
Dash lights (bit n = 1<<(n−1)): SHIFT 1, FULLBEAM 2, HANDBRAKE 3, PITSPEED 4, TC 5, SIGNAL_L 6, SIGNAL_R 7,
  SIGNAL_ANY 8, OILWARN 9, BATTERY 10, ABS 11, SPARE 12
```

## `Connect` / `Update(dt, vehicle) → working` / `Close`

See implementation.
