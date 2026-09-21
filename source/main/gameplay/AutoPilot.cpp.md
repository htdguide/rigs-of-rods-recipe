# source/main/gameplay/AutoPilot.cpp

> Proportional controllers for bank, vertical speed and airspeed; ILS localizer search; GPWS callouts.

**Needs** — [`AutoPilot.h`](AutoPilot.h.md) · [`Application.h`](../Application.h.md) · [`GameContext.h`](../GameContext.h.md) · [`physics/SimData.h`](../physics/SimData.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`gfx/GfxWater.h`](../gfx/GfxWater.h.md)
**Used by** — callers of [`AutoPilot.h`](AutoPilot.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

A simple, stable autopilot built from bounded proportional laws — enough to fly straight, hold altitude and speed, and follow an ILS to near the runway.

## State

See [`AutoPilot.h`](AutoPilot.h.md).

## `getAilerons`

```text
IF force_disabled: RETURN 0
bank = asin(clamp((right.y − left.y)/span, −1, 1)) in degrees (57.3)
WLV:   out = clamp(bank/100, ±0.5)
FIXED: track = atan2(v.x, −v.z)° from mean wingtip velocity; want = 2·wrap(track − heading) clamped ±45; out = clamp((bank − want)/100, ±0.5)
NAV:   intercept = runway_heading + clamp(hdev/10, ±1)·min(runway_distance/30, 60); then as FIXED toward intercept
last_aileron = (last_aileron + out)/2; RETURN last_aileron            # one-step smoothing
```

(The wrap only corrects values below −180.)

## `getElevator`

```text
wanted_vs = vs/196.87 (ft/min → m/s); current = mean wingtip v.y; pitch_rate = current − tail.v.y
VS mode, NAV heading (glideslope): ch = d·sin(vdev + 4°); oh = d·sin(4°)
    target = clamp((oh − ch)/5, ±5000 ft/min); out = clamp((target − current)/40 + pitch_rate/40, ±0.75)
VS mode otherwise: out = clamp((wanted_vs − current)/40 + pitch_rate/40, ±0.5)
FIXED alt: target = clamp((alt·0.3048 − mean wingtip y)/8, ±|wanted_vs|); out = clamp((target − current)/40 + pitch_rate/40, ±0.5)
```

Note FIXED altitude climbs/descends no faster than the VS setting (and holds level if VS is 0).

## `getThrottle(pilot, dt)`

Without IAS mode (or when force-disabled) returns the pilot's throttle. Otherwise indicated airspeed = ground speed (kt) × sqrt(ρ/1.225) with ISA density; throttle ramps ±dt/15 toward the target speed, clamped to 0.02..1.

## `UpdateIls`

```text
position = midpoint of the wingtips
FOR EACH terrain localizer (horizontal = runway direction beacon, vertical = glideslope beacon)
  diff = horizontal angle between the beacon's facing and the direction to the aircraft, wrapped to ±180
  IF |diff| < 80 (inside the beacon's cone)
    horizontal: nearest wins: hdev = diff; runway_heading = beacon angle − 90 (normalised to 0..360); runway_distance = distance
    vertical: nearest wins: vdev = elevation angle of the aircraft in the beacon's vertical plane − 4° glideslope
available flags; deviations default −90 when none
IF NAV AND GPWS AND the horizontal distance just crossed inward through 350 m (both > 10 m): "minimums" callout
IF NAV AND NOT force_disabled AND (within 20 m of either beacon OR ILS not available): wants_disconnect
```

## `gpws_update(spawn_height)` — only with sound enabled and GPWS on

Height above ground-or-sea (feet) at the cockpit minus the spawn height offset; when faster than ~10 kt, crossing downward through 100, 50, 40, 30, 20, 10 ft plays that callout. "Pull up" when descending faster than 10 (m/s × 1.9685) and ground contact would come within 10 s.
