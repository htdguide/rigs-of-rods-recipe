# source/main/gui/panels/GUI_VehicleInfoTPanel.cpp

> Tab contents, visibility policy, the damage statistics, and button-to-action mapping.

**Needs** — [`GUI_VehicleInfoTPanel.h`](GUI_VehicleInfoTPanel.h.md) · [`Application.h`](../../Application.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`physics/SimData.h`](../../physics/SimData.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`gameplay/Engine.h`](../../gameplay/Engine.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`gfx/GfxActor.h`](../../gfx/GfxActor.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_VehicleInfoTPanel.h`](GUI_VehicleInfoTPanel.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_VehicleInfoTPanel.h`](GUI_VehicleInfoTPanel.h.md).

## State

See header.

## Visibility

With "show vehicle buttons" on: the first time a vehicle is shown, post the hint "Hover the mouse on the left to see controls" and stay translucent for 5 s; afterwards translucent whenever the mouse is within the panel width of the left edge (and static menus are allowed). Opaque (hotkey) overrides. Placed at the left edge, 110 px below the top padding.

Header: vehicle preview (≤100×100) and colour-marked name; tabs Basics, Stats, Commands, Diag (a requested tab is selected once).

## Basics — one row per applicable action (icon, name, key binding button; lit when active)

Repair (reset on spot), pause actor physics; head lights (if head or tail lights), left/right blinkers, hazard (if any blinker), beacons (if beacon props), horn (trucks; police siren toggles, otherwise held while the button is pressed); custom lights 1–10 buttons (`CTRL+[1...]`); engine: ignition (running → contact off, else start), starter key hint while stopped, transfer case 4WD mode (if both 2WD and 4WD), transfer case ratio (if > 1), shift mode (cycles auto → semi-auto → manual → stick → ranges, with a console notice) and the relevant shift keys for the mode; axle / wheel differentials; traction control and ABS (unless hidden on the dashboard); parking brake (not for loads/boats/…); cruise control (with engine); lock (posts hook toggle + slide-node toggle linking requests) and secure (tie toggle); particles; mirrors (video cameras on/off); switch camera.

## Stats

- **Health** — `h = 10 × broken/beams + deformed/beams` (deformed = |L − L₀| > 0.0001 m, hydros excluded); shows health `(1 − h) × 100 %`, or "destruction 100 %" once h ≥ 1.
- Beam count; broken (count, %); deformed (count, %); average deformation (% of summed |L − L₀| per beam); average stress `1 − Σ|stress| / beams`.
- Nodes (and wheel nodes); total mass in kg, lb (×2.205) and tonnes (unit order follows the imperial setting).
- Trucks with engine: RPM / max (red over max), input shaft RPM, torque, power `rpm × (T + T × 6.8·psi/100) × π/30 / 1000` kW (and ×1.341 hp), gear, drive ratio, wheel and vehicle speed (km/h and mph; below 1 m/s shown as 0).
- Others: speed in knots, km/h (×1.852), mph (×1.151); aircraft altitude in feet and metres and per-engine RPM; boats per-screw throttle.
- Top speed; g-forces vertical / sagittal / lateral, current and max.

`UpdateStats` computes the beam sums from live data each frame (sim-synced).

## Commands

Vehicle description lines; the help image (80 px, or full-size 512×128 in a separate window when chosen or forced by setting); then every unique command key pair: description (or "~unlabeled~"), key 1 and key 2 buttons right-aligned (wrapping to a second line when narrow). Hovering a row highlights it and, in world space, draws the command's beams as thick green lines; pressing a key button makes it the active command key.

## Diag

Debug view radio: normal, skeleton, node details, beam details, and — when applicable — wheels, shocks, rotators, slide nodes, submeshes, buoyancy. For skeleton…beams: hide broken beams, beam stress, wheels, nodes; for detail views also hide wheel info.
