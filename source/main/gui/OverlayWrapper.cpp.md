# source/main/gui/OverlayWrapper.cpp

> Needle laws for the fallback HUDs, aspect-ratio correction, and aircraft panel mouse interaction.

**Needs** — [`OverlayWrapper.h`](OverlayWrapper.h.md) · [`physics/air/AeroEngine.h`](../physics/air/AeroEngine.h.md) · [`AppContext.h`](../AppContext.h.md) · [`gameplay/AutoPilot.h`](../gameplay/AutoPilot.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md) · [`gameplay/Character.h`](../gameplay/Character.h.md) · [`DashBoardManager.h`](DashBoardManager.h.md) · [`utils/ErrorUtils.h`](../utils/ErrorUtils.h.md) · [`physics/flex/FlexAirfoil.h`](../physics/flex/FlexAirfoil.h.md) · [`GameContext.h`](../GameContext.h.md) · [`gfx/GfxActor.h`](../gfx/GfxActor.h.md) · [`gfx/GfxScene.h`](../gfx/GfxScene.h.md) · [`utils/Language.h`](../utils/Language.h.md) · [`RoRVersion.h`](../../../source/version_info/RoRVersion.h.md) · [`physics/water/ScrewProp.h`](../physics/water/ScrewProp.h.md) · [`audio/SoundScriptManager.h`](../audio/SoundScriptManager.h.md) · [`terrain/Terrain.h`](../terrain/Terrain.h.md) · [`physics/air/TurboProp.h`](../physics/air/TurboProp.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine)
**Used by** — callers of [`OverlayWrapper.h`](OverlayWrapper.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`OverlayWrapper.h`](OverlayWrapper.h.md). All gauge inputs come from the sim buffers (snapshot), except the marine HUD, which reads the live actor.

## State

See header.

## Startup and aspect ratio

Look up every named overlay and element. The panels were authored for 4:3: at startup each element's height and top are multiplied by width/height once (a global "already scaled" flag stops repeats). Overlays loaded with auto-aspect get scale `s = (4/3)/(w/h)` (or `(3/4)/(h/w)` when that exceeds 1), scrolled by `(1 − s, s − 1)` to stay anchored bottom-right, re-applied on window resize. Throttle track geometry is taken from `thrusttrack1` and the throttle knob size.

## `showDashboardOverlays(show, actor)`

If the actor loaded its own HUD dashboard, toggle that instead and stop. Otherwise show the overlay matching the vehicle type (aircraft: dash + needles; boat: dash + needles; machine: dash with the vehicle's help material or black) or hide all.

## Land vehicle — `UpdateLandVehicleHUD`

- Gear text `g/N`, `N`, or `R`. Auto-shift letters: the selected one bright (red for N), others dim.
- Speedo needle angle `140 − |wheel speed| × 7 × 140 / speedo max kph`, floored at −140°.
- Tacho factor 0.072 (or `0.072 × 3500 / engine max RPM` when the vehicle's guisettings ask); angle `126 − |rpm| × factor`, clamped to [−120, 121]°.

## Tyre pressure — `UpdatePressureOverlay`

Shown only while pressurising; needle `135 − pressure × 2.7`°.

## Aircraft — `UpdateAerialHUD`

- Throttle knobs, fire lamps and ignition switches for up to 4 engines (hidden when absent).
- Indicated airspeed: ground speed in knots × √(ρ/1.225), with ρ from the standard atmosphere `p = 101325 (1 − 0.0065 h / 288.15)^5.24947`, `ρ = p × 1.20896e−5`. Needle: 0 below 23 kt; `(kt−23)/1.111` to 50; `24 + (kt−50)/0.8621` to 100; `82 + (kt−100)/0.8065` to 300; 329 beyond.
- Angle of attack from wing 4 (0 below 10 kt); drives the AOA sound modulation and starts the stall-warning sound above 18°; needle `−clamp(aoa, ±25) × 4.7 + 90`.
- Altimeter needle `height × 1.1811`°; text = height in hundreds of feet, 3 digits.
- Attitude: roll from the camera roll axis (flipped when inverted), pitch from the direction vector; the ADI tape scrolls by `−pitch × 0.25` and both ADI textures rotate by −roll.
- HSI: rose rotates with heading `atan2(dir·X, dir·−Z)`; bug offset by autopilot heading; ILS deviations clamped ±15, scrolled ×0.02.
- VVI (ft/min = vertical m/s × 196.85): linear 0.047°/fpm within ±1000, then `47 + (v−1000) × 0.01175` to ±6000, capped ±105.75; needle `90 − angle`.
- Engine RPM and (props only) torque %: `−5 + p × 1.9167` below 60 %, `110 + (p−60) × 4.075` below 110 %, else 314°. Prop pitch `value × 2`°.
- Autopilot lamps from modes; trim displays heading `%.3u`, altitude and V/S in hundreds (`+` for climb, `000` for zero), IAS `%.3u`.

**Notes** — "prop" is decided from engine 0's type and applied to all engines.

## Aircraft panel mouse input

Hover highlights a widget by switching its texture blend to additive with (.32, .30, .25). Left press on a hovered element: throttle knob starts a drag; engine-start switch flips ignition; autopilot switches toggle heading hold / wings level / nav, altitude hold / vertical speed, auto-throttle, GPWS, parking brake (0.2 s cooldown); trim buttons adjust heading ±1°, altitude ±100 ft, V/S ±100 fpm, IAS ±1 kt (0.1 s cooldown). While dragging, the throttle of the hovered engine = `1 − (mouse y − track top − knob offset)/track height`. Release ends the drag. Mouse events are only consumed while the aircraft panel is visible.

## Boat — `UpdateMarineHUD`

Throttle knobs at `track top + height × (0.5 − throttle/2)` for up to 2 screws; depth text `%2.1f` when 0.1–99.9 m above ground, else `--.-`; speed needle `knots × 4.2`°; rudder needle `rudder × 170`°.

## Race timer — `UpdateRacingGui`

Lap time `MM'SS.hh`; best time shown when set; lap time red when behind (diff > 0), green when ahead, white otherwise.

## Unused

`updateStats` (FPS/triangles/memory text into `Core/*` elements) is never called.
