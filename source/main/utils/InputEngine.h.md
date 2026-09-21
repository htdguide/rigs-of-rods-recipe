# source/main/utils/InputEngine.h

> Named game actions ("events"), their bindings to keys/buttons/axes, and the per-frame evaluation of each action's value.

**Needs** — [`Application.h`](../Application.h.md) · [`ForceFeedback.h`](ForceFeedback.h.md) · [Seam: Windowing and input devices](../../../SYSTEM-REQUIREMENTS.md#seam-windowing-and-input-devices)
**Used by** — [`AppContext.cpp`](../AppContext.cpp.md) · [`Application.cpp`](../Application.cpp.md) · [`GameContext.cpp`](../GameContext.cpp.md) · [`gameplay/Character.cpp`](../gameplay/Character.cpp.md) · [`gameplay/CruiseControl.cpp`](../gameplay/CruiseControl.cpp.md) · [`gameplay/Engine.cpp`](../gameplay/Engine.cpp.md) · [`gameplay/RepairMode.cpp`](../gameplay/RepairMode.cpp.md) · [`gameplay/Replay.cpp`](../gameplay/Replay.cpp.md) · [`gameplay/TyrePressure.cpp`](../gameplay/TyrePressure.cpp.md) · [`gfx/GfxActor.cpp`](../gfx/GfxActor.cpp.md) · [`gfx/camera/CameraManager.cpp`](../gfx/camera/CameraManager.cpp.md) · [`gui/GUIManager.cpp`](../gui/GUIManager.cpp.md) · [`gui/panels/GUI_GameChatBox.cpp`](../gui/panels/GUI_GameChatBox.cpp.md) · [`gui/panels/GUI_GameControls.cpp`](../gui/panels/GUI_GameControls.cpp.md) · [`gui/panels/GUI_GameControls.h`](../gui/panels/GUI_GameControls.h.md) · [`gui/panels/GUI_MainSelector.cpp`](../gui/panels/GUI_MainSelector.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](../gui/panels/GUI_SurveyMap.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`main.cpp`](../main.cpp.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorSpawner.cpp`](../physics/ActorSpawner.cpp.md) · [`physics/Savegame.cpp`](../physics/Savegame.cpp.md) · [`physics/SimData.h`](../physics/SimData.h.md) · [`scripting/ScriptEngine.cpp`](../scripting/ScriptEngine.cpp.md) · [`scripting/bindings/InputEngineAngelscript.cpp`](../scripting/bindings/InputEngineAngelscript.cpp.md) · [`terrain/TerrainEditor.cpp`](../terrain/TerrainEditor.cpp.md) · [`ForceFeedback.cpp`](ForceFeedback.cpp.md) · [`InputEngine.cpp`](InputEngine.cpp.md)
**Tier floor** — T2

## Purpose

Game code never asks "is W down"; it asks "what is the value of `TRUCK_ACCELERATE`", a float in 0..1. This file declares the fixed set of ~320 actions, the binding record, and the engine that owns input devices and evaluates actions. Bindings come from text map files so users and controller vendors can ship mappings. Implementation, the map-file grammar and the default bindings: [`InputEngine.cpp`](InputEngine.cpp.md).

## State

```text
ENUM TriggerType = NONE | KEYBOARD | MOUSE_BUTTON | MOUSE_AXIS_X | MOUSE_AXIS_Y | MOUSE_AXIS_Z
                 | JOY_BUTTON | JOY_AXIS_ABS | JOY_AXIS_REL | JOY_POV | JOY_SLIDER_X | JOY_SLIDER_Y

ENUM InputSource = ANY | DIGITAL | ANALOG

ENUM Event = 0 .. EV_MODE_LAST-1       # ~320 actions, grouped by name prefix:
  AIRPLANE_* BOAT_* SKY_* CAMERA_* CHARACTER_* COMMANDS_01..84 COMMON_* GRASS_* MENU_*
  SURVEY_MAP_* TRUCK_* COMMON_QUICKSAVE_01..10 COMMON_QUICKLOAD_01..10 TRUCKEDIT_RELOAD ROAD_EDITOR_*
  # numbering is internal; files and scripts refer to events by NAME (the enum name without "EV_")

RECORD Trigger                          # one binding
  type          : TriggerType
  device_config : int                   # which map file defined it: -1 input.map, -2 built-in default, >=0 joystick index
  # keyboard
  key           : key code
  explicit      : bool                  # modifiers must match exactly
  ctrl, shift, alt : bool
  # joystick
  joystick, button, axis, pov, pov_direction, slider : int
  deadzone      : real = 0.1
  linearity     : real = 1.0
  region        : int                   # 0 full axis, +1 upper half, -1 lower half
  reverse, half, use_digital : bool
  slider_reverse : bool
  # book-keeping for the controls editor
  configline    : text                  # original option text (key combo / button / direction / axis options)
  group         : text                  # event-name prefix before the first '_' (e.g. "TRUCK")
  comments      : text                  # ';' comment lines that preceded it in the file

RECORD InputEngine
  devices       : keyboard, mouse, up to 10 joysticks, optional force-feedback interface (from the first joystick that has one)
  key_state     : map<key code, bool>   # RoR's own view, fed by events (not the device's)
  joy_state     : per-joystick snapshot (axes, buttons, POVs, sliders)
  mouse_state   : position + RoR-tracked button bits
  bindings      : map<Event, list<Trigger>>
  simulated     : map<Event, real>      # injected values (scripts, touch UI) override devices
  suppressed    : map<Event, bool>      # forced to 0
  bounce_timer  : map<Event, real>      # seconds left before a "bounced" query may fire again; negative keys = raw keys
  loaded_files  : per-joystick map filename
  focus_workaround_frames, focus_workaround_lmb_downs : int
```

Limits: 10 joysticks, 4 POVs and 4 sliders per joystick, 32 axes.

## Setup and device listeners

`SetKeyboardListener`, `SetMouseListener`, `SetJoystickListener`, `destroy`, and the constructor, which opens devices and loads bindings — see `.cpp`.

## Input processing

`Capture`, `updateKeyBounces(dt)`, `processMouseMotionEvent`, `processMousePressEvent`, `processMouseReleaseEvent`, `ProcessKeyPress`, `ProcessKeyRelease`, `ProcessJoystickEvent`, `resetKeysAndMouseButtons`, `setEventSimulatedValue`, `setEventStatusSupressed`.

## Event queries

`getEventValue(event, pure, source)`, `getEventBoolValue`, `getEventBoolValueBounce(event, time=0.2)`, `getEventBounceTime`, `isEventAnalog`, `isEventDefined`, `isKeyDown` (device), `isKeyDownEffective` (RoR's view), `isKeyDownValueBounce`.

## Binding management

`loadConfigFile(device)`, `saveConfigFile(device)`, `getLoadedConfigFile`, `processLine(text, device)`, `updateConfigline(trigger)`, `addEvent`, `addEventDefault`, `updateEvent`, `eraseEvent`, `clearEvents`, `clearEventsByDevice`, `clearAllEvents`.

## Descriptions and helpers

`getEventCommand`, `getEventCommandTrimmed` (drops `EXPL+`), `getEventConfig`, `getEventDefaultConfig`, `getKeyForCommand`, `getKeboardKeyForCommand`, `getTriggerCommand`, `getDeviceName`, `getModifierKeyName`, `getKeyNameForKeyCode`, `getCurrentKeyCombo`, `getCurrentJoyButton`, `getCurrentPovValue`, `getJoyComponentCount`, `getJoyVendor`, `getNumJoysticks`, `getMouseState`, `getMouseNormalizedScreenPos`, `getEventTypeName`, `resolveEventName`, `eventIDToName`, `eventIDToDescription`, `windowResized`, `getForceFeedbackDevice`.
