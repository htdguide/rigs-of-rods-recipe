# source/main/gui/DashBoardManager.h

> Per-vehicle instrument data (a typed slot per gauge input) and the data-driven dashboard layouts that display it.

**Needs** — [`Application.h`](../Application.h.md) · [`utils/memory/RefCountingObject.h`](../utils/memory/RefCountingObject.h.md) · [`utils/memory/RefCountingObjectPtr.h`](../utils/memory/RefCountingObjectPtr.h.md) · [`RTTLayer.h`](RTTLayer.h.md) · [Seam: Retained layout GUI](../../../SYSTEM-REQUIREMENTS.md#seam-retained-layout-gui)
**Used by** — [`GameContext.cpp`](../GameContext.cpp.md) · [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`DashBoardManager.cpp`](DashBoardManager.cpp.md) · [`OverlayWrapper.cpp`](OverlayWrapper.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](panels/GUI_TopMenubar.cpp.md) · [`network/OutGauge.cpp`](../network/OutGauge.cpp.md) · [`physics/Actor.cpp`](../physics/Actor.cpp.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`resources/rig_def_fileformat/RigDef_Parser.cpp`](../resources/rig_def_fileformat/RigDef_Parser.cpp.md) · [`scripting/bindings/DashBoardManagerAngelscript.cpp`](../scripting/bindings/DashBoardManagerAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

Decouples the simulation from instrument artwork. The actor writes named values (rpm, gear, lamp states…) into slots every frame; dashboards are content-authored GUI layouts whose widgets declare, in their user attributes, which slot drives them and how. The same layout can render on screen (HUD) or into a texture shown on a 3D cockpit mesh. Implementation: [`DashBoardManager.cpp`](DashBoardManager.cpp.md).

## State

```text
CONSTANTS: text slot 255 chars; up to 6 screwprops, 6 aero engines, 6 wings; ≤10 geometric animations per widget
ENUM DataType: BOOL, INT, FLOAT, CHAR (text), INVALID
RECORD DashSlot: type, bool, int, float, text[255], enabled, name
ENUM DashData — built-in slot ids, in this order (names are the layout "link" names):
  rpm, speedo_kph, speedo_mph, engine_turbo, engine_ignition, engine_running, engine_battery, engine_clutch_warning,
  engine_gear, engine_num_gear, engine_gear_string ("<g>/<max>"), engine_autogear_string ("P R N G …"), engine_auto_gear,
  engine_clutch, brake, accelerator, roll, roll_corr, roll_corr_active, pitch, parkingbrake, locked, low_pressure,
  tractioncontrol_mode, antilockbrake_mode, ties_mode,
  screw_throttle_0..5, screw_steer_0..5, water_depth, water_speed (knots),
  aeroengine_throttle_0..5, aeroengine_failed_0..5, aeroengine_rpm_0..5, airspeed, wing_aoa_0..5,
  altitude, altitude_string, odometer_total, odometer_user,
  custom_light1..10, headlights, highbeams, foglights, sidelights, brake_lights, reverse_light, beacons,
  lights (legacy alias of sidelights), signal_turnleft, signal_turnright, signal_warning,
  guisetting_speedo_tex, guisetting_tacho_tex, guisetting_help (text: textures from the vehicle's guisettings),
  guisetting_speedo_kph, guisetting_tacho_rpm (float: like speedo_kph / rpm, but their gauge range follows guisettings)
  MAX — custom inputs are numbered from here
FLAGS LoadDashBoard: SCREEN_HUD, RTT_TEXTURE (drawn to texture), RENDERDASH (classic render-to-texture dash), STACKABLE
RECORD DashBoardManager (reference-counted, per actor)
  slots : list<DashSlot>          # index = DashData id or custom id
  dashboards : list<DashBoard>; hud loaded, rtt loaded flags; custom input count; loaded RTT count; visible; actor
RECORD DashBoard
  layout file, widgets, main widget ("_Main"), unique name prefix, render-to-texture layer or none, visible
  controls : list<Control>
RECORD Control
  widget, name, visibility slot, initial size and position, last lamp state
  graphical animation (≤1): slot, kind (series | textcolor | textformat | textstring | lamp | imagetexture),
       condition (none | > x | < x), texture, printf format, "negative zero" rendering of the format, last value
  geometric animations (≤10): slot, kind (rotate | scale | translate), widget min/max, value min/max, direction, last value
```

Not thread-safe by design: written by the sim thread's post-step, read by the GUI in the same frame.

## API

Slots: `registerCustomInput(name, type) → id | −1`, `_getBool/_getInt/_getFloat`, `getNumeric` (bool→0/1, int, float; text→0), `getChar`, `getEnabled`, `set*`, `setEnabled`, `getDataType`, `getLinkIDForName`, `getLinkNameForID`, `getInputCount`. Out-of-range ids read as false/0/none and ignore writes (except the raw int/float getters).
Dashboards: `loadDashBoard(file, flags, rtt layer) → dashboard`, `update(dt)`, `updateFeatures`, `setVisible` (screen ones), `setVisible3d` (texture ones), `windowResized`, `wasDashboardHudLoaded`, `wasDashboardRttLoaded`, `getNumLoadedRTTDashboards`.
