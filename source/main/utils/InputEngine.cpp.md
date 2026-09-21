# source/main/utils/InputEngine.cpp

> Device setup, the `.map` binding grammar, action evaluation (dead zones, regions, modifiers), and the built-in default bindings.

**Needs** — [`InputEngine.h`](InputEngine.h.md) · [`Application.h`](../Application.h.md) · [`AppContext.h`](../AppContext.h.md) · [`system/Console.h`](../system/Console.h.md) · [`Language.h`](Language.h.md) · [Seam: Windowing and input devices](../../../SYSTEM-REQUIREMENTS.md#seam-windowing-and-input-devices)
**Used by** — callers of [`InputEngine.h`](InputEngine.h.md) (see its Used by)
**Tier floor** — T2

## Purpose

Everything a player's controller setup depends on. The map-file format is user data (`config/input.map` plus per-device files) and must be read exactly as described here.

## State

See header twin.

## Construction and device setup

**Contract** — opens the input system on the render window, creates keyboard (Unicode text translation on), up to 10 joysticks, and the mouse; errors creating any device are logged and that device is absent. The first joystick exposing force feedback supplies the FFB device. Then bindings are loaded (below).

Grab behaviour depends on `io_input_grab_mode`: unless it is ALL, devices are opened non-exclusive and foreground-only (the OS cursor and other windows keep working), with key auto-repeat off and the OS cursor hidden. On Windows the mouse is initially centred.

```text
FUNCTION load_bindings()
  load_config_file(-1)                      # config/input.map
  FOR EACH joystick j: load_config_file(j)  # per-device file
  complete_missing_events()                 # built-in defaults for anything still unbound

FUNCTION load_config_file(device)
  IF device == -1: file = "input.map"
  ELSE
    name = joystick vendor string with each of  \ / space # @ ? ! $ % ^ & * ( ) + = - > < . : ' | " ;  replaced by '_'
    file = name + ".windows.map" / ".linux.map" if that exists in the config group, else name + ".map"
  FOR EACH line IN file (lines of 5 or fewer chars are skipped)
    process_line(line, device)

FUNCTION complete_missing_events()
  FOR EACH event E in the built-in table
    IF E has no entry at all in bindings (not even an explicit "None")
      process_line(name(E) + " " + default(E), device = -2)
```

**Notes** — "no entry" versus "entry with no trigger" matters: a user file line `TRUCK_HORN None` deliberately unbinds the horn and stops the default from coming back.

## Map-file grammar

Each non-comment line: `<EVENT_NAME> <DeviceType> <arguments…>`, whitespace separated. Lines starting with `;` are comments; consecutive comments are attached to the next binding (kept for the editor).

| DeviceType | Arguments | Meaning |
|---|---|---|
| `Keyboard` | `[MOD+]…KEY` | `+`-joined; modifiers `SHIFT`, `CTRL`, `ALT`, `EXPL`; the **last** token is the key name (list below) |
| `JoystickButton` | `<joy> <button>` | |
| `JoystickAxis` | `<joy> <axis> [OPT+OPT…]` | options: `HALF`, `REVERSE`, `LINEAR`, `UPPER`, `LOWER`, `RELATIVE`, `DIGITAL`, `DEADZONE=<f>`, `LINEARITY=<f>` |
| `JoystickPov` | `<joy> <pov> <dir>` | dir: `North South East West NorthEast SouthEast NorthWest SouthWest` |
| `JoystickSlider` | `<joy> <X\|Y> <slider> [REVERSE]` | |
| `None` | — | event exists but is unbound |
| `MouseButton`, `MouseAxisX/Y/Z` | — | recognised but ignored (no mouse bindings) |

Rules: an unknown event name is logged and the line ignored; an unknown key name gives an unassigned key. In a **per-device** file the `<joy>` number is ignored and replaced by that device's index, so vendor files work whatever slot the device lands in. Defaults: dead zone 0.1, linearity 1.0.

**Saving** writes every binding whose `device_config` matches, one per line: event name left-padded to 35 columns, device type to 20, then the config string; joystick lines get a dummy `0` device number. `updateConfigline` rebuilds the option text for axes (`REVERSE`, `UPPER`/`LOWER`, `DEADZONE=%.2f` if ≠ 0.1, `LINEARITY=%.2f` if ≠ 1.0) and sliders (`REVERSE`, region) after the editor changes them.

## `getEventValue(event, pure = false, source = ANY)`

**Contract** — the action's current value, normally 0..1 (relative axes may exceed). Highest value over all triggers wins, so a keyboard key and a pedal can drive the same action.

```text
FUNCTION event_value(E, pure, source)
  IF simulated[E] != 0: RETURN simulated[E]
  IF suppressed[E]: RETURN 0
  best = 0
  FOR EACH t IN bindings[E]
    v = 0
    IF source IN {ANY, DIGITAL}
      KEYBOARD:   IF key_state[t.key]:
                    IF t.explicit: require ctrl/shift/alt state to EQUAL t's flags   # "F1" vs "SHIFT+F1" are distinct
                    ELSE:          require each modifier t names to be held
                    v = 1
      MOUSE_BUTTON: v = left button down                 # all mouse bindings read the left button (unfinished feature)
      JOY_BUTTON: v = button state (0 if joystick/button missing, logs)
      JOY_POV:    v = 1 if pov direction bits contain t.pov_direction
    IF source IN {ANY, ANALOG}
      MOUSE_AXIS_*: v = mouse abs coordinate / 32767
      JOY_AXIS_REL: v = axis.rel / MAX_AXIS
      JOY_AXIS_ABS: v = shape_axis(axis.abs / MAX_AXIS, t, pure)
      JOY_SLIDER_X/Y: v = (slider / MAX_AXIS + 1) / 2; IF t.slider_reverse: v = 1 - v
    best = max(best, v)
  RETURN best

FUNCTION shape_axis(raw in [-1, 1], t, pure)
  v = CASE t.region OF
        0:  (raw + 1) / 2                    # full travel -> 0..1
        -1: IF raw > 0 THEN 0 ELSE -raw      # lower half only
        +1: IF raw < 0 THEN 0 ELSE raw       # upper half only
  IF t.half
    v = (1 + v) / 2                          # no dead zone on half axes
    IF t.reverse: v = 1 - v
    IF NOT pure: v = v * t.linearity
  ELSE
    IF t.reverse: v = 1 - v
    IF NOT pure: v = dead_zone(v, t.deadzone); v = v * t.linearity
  IF t.use_digital: v = 1 IF v >= 0.5 ELSE 0
  RETURN v

FUNCTION dead_zone(v, dz)
  IF dz < 0.0001: RETURN v
  IF |v| < dz: RETURN 0
  RETURN (v - dz) / (1 - dz)                 # rescale the remainder to reach 1 at full travel
```

**Notes** — "linearity" is a plain gain, not a curve (the name is historical). `pure` skips dead zone and gain; it exists so `isEventAnalog` can tell a connected axis from a disconnected one. A missing joystick or component yields 0 rather than an error because maps are routinely shared between users with different hardware.

## `isEventAnalog(E)`

**Contract** — true if any trigger of E is an axis/slider type **and** its pure analog value is non-zero right now (i.e. the device is present and not resting at exactly 0). Vehicle code uses this to switch between analog steering and keyboard-rate steering.

## Bounce queries

**Contract** — `getEventBoolValue(E)` is `value > 0.5`. `getEventBoolValueBounce(E, t = 0.2)` returns true at most once per `t` seconds while held: if the timer is running it returns false; otherwise it evaluates, and on true arms the timer. `updateKeyBounces(dt)` counts all timers down each frame. `isKeyDownValueBounce(key, t)` is the same for a raw key, using the negated key code as the timer key.

## Key and mouse state

**Contract** — `ProcessKeyPress/Release` set RoR's own `key_state`; `ProcessJoystickEvent` stores the device snapshot (invalid device ids are treated as 0). Mouse motion updates only the position; press/release update only the one button bit. `resetKeysAndMouseButtons` clears all key states and mouse buttons and restarts the focus workaround.

**Notes** — the focus workaround: after the window regains focus, the device layer fabricates a left-button press with no release. A left press is therefore ignored if it is the first one since the reset *and* no frame has been captured since the reset (`Capture` increments the frame counter). RoR keeps its own button bits rather than trusting the device's state for the same reason.

## Descriptive helpers

**Contract** —
- `getEventCommand` / `getEventConfig` show the first binding: keyboard → the key combo text; button → number; POV → `"<pov> <dir>"`; slider → `"X|Y <n>"` (+ options for config); axis → `"<axis>"` (+ options).
- `getCurrentKeyCombo(out)` builds `"CTRL+SHIFT+F1"`-style text from held keys, modifiers first (left/right collapsed to `SHIFT`, `CTRL`, `ALT`); returns the number of non-modifier keys, or minus the modifier count if only modifiers are held, or 0 with "(Please press a key)". Used by the key-capture dialog.
- `getCurrentJoyButton` / `getCurrentPovValue` return the first pressed button / non-centred POV across all joysticks (for binding capture).
- `getMouseNormalizedScreenPos` — mouse position as 0..1 of the window.
- `getModifierKeyName` — localized "Left Alt", "Right Ctrl", etc.
- `resolveEventName`, `eventIDToName`, `eventIDToDescription` — linear lookups in the table below; unknown names return −1 / `"unknown"`.
- `getEventGroup(name)` — text before the first `_`.

## Built-in default bindings

`EXPL+` marks an explicit combination (modifiers must match exactly). Empty = unbound by default.

| Event | Default binding | Meaning |
|---|---|---|
| `COMMON_ACCELERATE_SIMULATION` | `Keyboard CTRL+EQUALS` | accelerate the simulation |
| `COMMON_DECELERATE_SIMULATION` | `Keyboard SHIFT+EQUALS` | decelerate the simulation |
| `COMMON_RESET_SIMULATION_PACE` | `Keyboard BACKSLASH` | reset the simulation pace |
| `COMMON_OUTPUT_POSITION` | `Keyboard H` | write current position to log (you can open the logfile and reuse the position) |
| `COMMON_QUIT_GAME` | `Keyboard EXPL+ESCAPE` | exit the game |
| `COMMON_QUICKLOAD` | `Keyboard MULTIPLY` | quickload scene |
| `COMMON_QUICKSAVE` | `Keyboard DIVIDE` | quicksave scene |
| `COMMON_SCREENSHOT` | `Keyboard EXPL+SYSRQ` | take a screenshot |
| `COMMON_SCREENSHOT_BIG` | `Keyboard EXPL+CTRL+SYSRQ` | take a big screenshot (3 times the screen size) |
| `COMMON_TOGGLE_MAT_DEBUG` | — | debug purpose - dont use |
| `COMMON_TOGGLE_PHYSICS` | `Keyboard EXPL+J` | enable or disable physics |
| `COMMON_FOV_LESS` | `Keyboard EXPL+NUMPAD7` | decreases the current FOV value |
| `COMMON_FOV_MORE` | `Keyboard EXPL+CTRL+NUMPAD7` | increase the current FOV value |
| `COMMON_FOV_RESET` | `Keyboard EXPL+SHIFT+NUMPAD7` | reset the FOV value |
| `COMMON_SAVE_TERRAIN` | `Keyboard EXPL+ALT+SHIF+CTRL+M` | save the currently loaded terrain to a mesh file |
| `COMMON_TOGGLE_TERRAIN_EDITOR` | `Keyboard EXPL+SHIFT+Y` | toggle terrain editor |
| `COMMON_FULLSCREEN_TOGGLE` | `Keyboard EXPL+ALT+RETURN` | toggle between windowed and fullscreen mode |
| `COMMON_ENTER_OR_EXIT_TRUCK` | `Keyboard RETURN` | enter or exit a truck |
| `COMMON_ENTER_NEXT_TRUCK` | `Keyboard EXPL+CTRL+RBRACKET` | enter next truck |
| `COMMON_ENTER_PREVIOUS_TRUCK` | `Keyboard EXPL+CTRL+LBRACKET` | enter previous truck |
| `COMMON_REMOVE_CURRENT_TRUCK` | `Keyboard EXPL+CTRL+DELETE` | remove current truck |
| `COMMON_TRUCK_REMOVE` | `Keyboard EXPL+CTRL+SHIFT+DELETE` | delete current truck |
| `COMMON_RESPAWN_LAST_TRUCK` | `Keyboard EXPL+CTRL+PERIOD` | respawn last truck |
| `COMMON_GET_NEW_VEHICLE` | `Keyboard EXPL+CTRL+G` | get new vehicle |
| `COMMON_PRESSURE_LESS` | `Keyboard LBRACKET` | decrease tire pressure (note: only very few trucks support this) |
| `COMMON_PRESSURE_MORE` | `Keyboard RBRACKET` | increase tire pressure (note: only very few trucks support this) |
| `COMMON_LOCK` | `Keyboard EXPL+L` | connect hook node to a node in close proximity |
| `COMMON_AUTOLOCK` | `Keyboard EXPL+ALT+L` | unlock autolock hook node |
| `COMMON_ROPELOCK` | `Keyboard EXPL+CTRL+L` | connect a rope to a node in close proximity |
| `COMMON_REPAIR_TRUCK` | `Keyboard BACK` | repair truck |
| `COMMON_LIVE_REPAIR_MODE` | `Keyboard ALT+BACK` | toggle truck interactive repair mode |
| `COMMON_RESCUE_TRUCK` | `Keyboard EXPL+R` | teleport to rescue truck |
| `COMMON_RESET_TRUCK` | `Keyboard I` | reset truck to original starting position |
| `COMMON_TOGGLE_RESET_MODE` | `Keyboard EXPL+APOSTROPHE` | toggle reset mode |
| `COMMON_SECURE_LOAD` | `Keyboard O` | tie a load to the truck |
| `COMMON_TOGGLE_TRUCK_BEACONS` | `Keyboard M` | toggle truck beacons |
| `COMMON_TOGGLE_TRUCK_LOW_BEAMS` | `Keyboard EXPL+N` | toggle truck low beams |
| `COMMON_CYCLE_TRUCK_LIGHTS` | `Keyboard EXPL+CTRL+N` | cycle between light modes |
| `COMMON_TOGGLE_TRUCK_HIGH_BEAMS` | `Keyboard EXPL+SHIFT+N` | toggle truck high beams |
| `COMMON_TOGGLE_TRUCK_FOG_LIGHTS` | `Keyboard EXPL+ALT+N` | toggle truck fog lights |
| `COMMON_TOGGLE_CUSTOM_PARTICLES` | `Keyboard G` | toggle particle cannon |
| `COMMON_TOGGLE_REPLAY_MODE` | `Keyboard EXPL+CTRL+J` | enable or disable replay mode |
| `COMMON_REPLAY_FORWARD` | `Keyboard EXPL+RIGHT` | more replay forward |
| `COMMON_REPLAY_BACKWARD` | `Keyboard EXPL+LEFT` | more replay backward |
| `COMMON_REPLAY_FAST_FORWARD` | `Keyboard EXPL+SHIFT+RIGHT` | move replay fast forward |
| `COMMON_REPLAY_FAST_BACKWARD` | `Keyboard EXPL+SHIFT+LEFT` | move replay fast backward |
| `COMMON_CONSOLE_TOGGLE` | `Keyboard EXPL+GRAVE` | show / hide the console |
| `COMMON_ENTER_CHATMODE` | `Keyboard EXPL+Y` | enter the chat |
| `COMMON_SEND_CHAT` | `Keyboard RETURN` | sends the entered text |
| `COMMON_HIDE_GUI` | `Keyboard EXPL+U` | hide all GUI elements |
| `COMMON_TOGGLE_DASHBOARD` | `Keyboard EXPL+CTRL+U` | display or hide the dashboard overlay |
| `COMMON_TOGGLE_DEBUG_VIEW` | `Keyboard EXPL+K` | toggle debug view |
| `COMMON_CYCLE_DEBUG_VIEWS` | `Keyboard EXPL+CTRL+K` | cycle debug views |
| `COMMON_TRUCK_INFO` | `Keyboard EXPL+T` | toggle truck HUD |
| `COMMON_TRUCK_DESCRIPTION` | `Keyboard EXPL+CTRL+T` | toggle truck description |
| `COMMON_NETCHATDISPLAY` | `Keyboard EXPL+SHIFT+U` | display or hide net chat |
| `COMMON_NETCHATMODE` | `Keyboard EXPL+CTRL+U` | toggle between net chat display modes |
| `COMMON_TOGGLE_STATS` | `Keyboard EXPL+F` | toggle Ogre statistics (FPS etc.) |
| `COMMON_QUICKSAVE_01` | `Keyboard EXPL+ALT+CTRL+1` | save scene in slot 01 |
| `COMMON_QUICKSAVE_02` | `Keyboard EXPL+ALT+CTRL+2` | save scene in slot 02 |
| `COMMON_QUICKSAVE_03` | `Keyboard EXPL+ALT+CTRL+3` | save scene in slot 03 |
| `COMMON_QUICKSAVE_04` | `Keyboard EXPL+ALT+CTRL+4` | save scene in slot 04 |
| `COMMON_QUICKSAVE_05` | `Keyboard EXPL+ALT+CTRL+5` | save scene in slot 05 |
| `COMMON_QUICKSAVE_06` | `Keyboard EXPL+ALT+CTRL+6` | save scene in slot 06 |
| `COMMON_QUICKSAVE_07` | `Keyboard EXPL+ALT+CTRL+7` | save scene in slot 07 |
| `COMMON_QUICKSAVE_08` | `Keyboard EXPL+ALT+CTRL+8` | save scene in slot 08 |
| `COMMON_QUICKSAVE_09` | `Keyboard EXPL+ALT+CTRL+9` | save scene in slot 09 |
| `COMMON_QUICKSAVE_10` | `Keyboard EXPL+ALT+CTRL+0` | save scene in slot 10 |
| `COMMON_QUICKLOAD_01` | `Keyboard EXPL+ALT+1` | load scene from slot 01 |
| `COMMON_QUICKLOAD_02` | `Keyboard EXPL+ALT+2` | load scene from slot 02 |
| `COMMON_QUICKLOAD_03` | `Keyboard EXPL+ALT+3` | load scene from slot 03 |
| `COMMON_QUICKLOAD_04` | `Keyboard EXPL+ALT+4` | load scene from slot 04 |
| `COMMON_QUICKLOAD_05` | `Keyboard EXPL+ALT+5` | load scene from slot 05 |
| `COMMON_QUICKLOAD_06` | `Keyboard EXPL+ALT+6` | load scene from slot 06 |
| `COMMON_QUICKLOAD_07` | `Keyboard EXPL+ALT+7` | load scene from slot 07 |
| `COMMON_QUICKLOAD_08` | `Keyboard EXPL+ALT+8` | load scene from slot 08 |
| `COMMON_QUICKLOAD_09` | `Keyboard EXPL+ALT+9` | load scene from slot 09 |
| `COMMON_QUICKLOAD_10` | `Keyboard EXPL+ALT+0` | load scene from slot 10 |
| `TRUCK_ACCELERATE` | `Keyboard UP` | accelerate the truck |
| `TRUCK_ACCELERATE_MODIFIER_25` | `Keyboard ALT+UP` | accelerate with 25 percent pedal input |
| `TRUCK_ACCELERATE_MODIFIER_50` | `Keyboard CTRL+UP` | accelerate with 50 percent pedal input |
| `TRUCK_BLINK_LEFT` | `Keyboard EXPL+COMMA` | toggle left direction indicator (blinker) |
| `TRUCK_BLINK_RIGHT` | `Keyboard EXPL+PERIOD` | toggle right direction indicator (blinker) |
| `TRUCK_BLINK_WARN` | `Keyboard EXPL+MINUS` | toggle all direction indicators |
| `TRUCK_BRAKE` | `Keyboard DOWN` | brake |
| `TRUCK_BRAKE_MODIFIER_25` | `Keyboard ALT+DOWN` | brake with 25 percent pedal input |
| `TRUCK_BRAKE_MODIFIER_50` | `Keyboard CTRL+DOWN` | brake with 50 percent pedal input |
| `TRUCK_HORN` | `Keyboard H` | truck horn |
| `TRUCK_LIGHTTOGGLE1` | `Keyboard EXPL+CTRL+1` | toggle custom light 1 |
| `TRUCK_LIGHTTOGGLE2` | `Keyboard EXPL+CTRL+2` | toggle custom light 2 |
| `TRUCK_LIGHTTOGGLE3` | `Keyboard EXPL+CTRL+3` | toggle custom light 3 |
| `TRUCK_LIGHTTOGGLE4` | `Keyboard EXPL+CTRL+4` | toggle custom light 4 |
| `TRUCK_LIGHTTOGGLE5` | `Keyboard EXPL+CTRL+5` | toggle custom light 5 |
| `TRUCK_LIGHTTOGGLE6` | `Keyboard EXPL+CTRL+6` | toggle custom light 6 |
| `TRUCK_LIGHTTOGGLE7` | `Keyboard EXPL+CTRL+7` | toggle custom light 7 |
| `TRUCK_LIGHTTOGGLE8` | `Keyboard EXPL+CTRL+8` | toggle custom light 8 |
| `TRUCK_LIGHTTOGGLE9` | `Keyboard EXPL+CTRL+9` | toggle custom light 9 |
| `TRUCK_LIGHTTOGGLE10` | `Keyboard EXPL+CTRL+0` | toggle custom light 10 |
| `TRUCK_PARKING_BRAKE` | `Keyboard P` | toggle parking brake |
| `TRUCK_TRAILER_PARKING_BRAKE` | `Keyboard EXPL+CTRL+P` | toggle trailer parking brake |
| `TRUCK_ANTILOCK_BRAKE` | `Keyboard EXPL+SHIFT+B` | toggle antilock brake |
| `TRUCK_TOGGLE_VIDEOCAMERA` | `Keyboard EXPL+CTRL+V` | toggle videocamera |
| `TRUCK_TRACTION_CONTROL` | `Keyboard EXPL+SHIFT+T` | toggle traction control |
| `TRUCK_CRUISE_CONTROL` | `Keyboard EXPL+SPACE` | toggle cruise control |
| `TRUCK_CRUISE_CONTROL_READJUST` | `Keyboard EXPL+CTRL+SPACE` | match target speed / rpm with current truck speed / rpm |
| `TRUCK_CRUISE_CONTROL_ACCL` | `Keyboard EXPL+CTRL+R` | increase target speed / rpm |
| `TRUCK_CRUISE_CONTROL_DECL` | `Keyboard EXPL+CTRL+F` | decrease target speed / rpm |
| `TRUCK_STARTER` | `Keyboard S` | hold to start the engine |
| `TRUCK_STEER_LEFT` | `Keyboard LEFT` | steer left |
| `TRUCK_STEER_RIGHT` | `Keyboard RIGHT` | steer right |
| `TRUCK_TOGGLE_CONTACT` | `Keyboard X` | toggle ignition |
| `TRUCK_TOGGLE_FORWARDCOMMANDS` | `Keyboard EXPL+CTRL+SHIFT+F` | toggle forwardcommands |
| `TRUCK_TOGGLE_IMPORTCOMMANDS` | `Keyboard EXPL+CTRL+SHIFT+I` | toggle importcommands |
| `TRUCK_TOGGLE_PHYSICS` | `Keyboard END` | toggle physics |
| `TRUCK_TOGGLE_INTER_AXLE_DIFF` | `Keyboard EXPL+ALT+W` | cycle between available inter axle differential modes |
| `TRUCK_TOGGLE_INTER_WHEEL_DIFF` | `Keyboard EXPL+W` | cycle between available inter wheel differential modes |
| `TRUCK_TOGGLE_TCASE_4WD_MODE` | `Keyboard EXPL+CTRL+W` | toggle transfer case mode |
| `TRUCK_TOGGLE_TCASE_GEAR_RATIO` | `Keyboard EXPL+SHIFT+W` | toggle transfer case gear ratio |
| `TRUCK_LEFT_MIRROR_LEFT` | `Keyboard EXPL+SEMICOLON` | move left mirror to the left |
| `TRUCK_LEFT_MIRROR_RIGHT` | `Keyboard EXPL+CTRL+SEMICOLON` | move left mirror to the right |
| `TRUCK_RIGHT_MIRROR_LEFT` | `Keyboard EXPL+COLON` | more right mirror to the left |
| `TRUCK_RIGHT_MIRROR_RIGHT` | `Keyboard EXPL+CTRL+COLON` | move right mirror to the right |
| `TRUCK_AUTOSHIFT_DOWN` | `Keyboard PGDOWN` | shift automatic transmission one gear down |
| `TRUCK_AUTOSHIFT_UP` | `Keyboard PGUP` | shift automatic transmission one gear up |
| `TRUCK_MANUAL_CLUTCH` | `Keyboard LSHIFT` | manual clutch (for manual transmission) |
| `TRUCK_MANUAL_CLUTCH_MODIFIER_25` | `Keyboard ALT+LSHIFT` | manual clutch with 25 percent pedal input |
| `TRUCK_MANUAL_CLUTCH_MODIFIER_50` | `Keyboard CTRL+LSHIFT` | manual clutch with 50 percent pedal input |
| `TRUCK_SHIFT_DOWN` | `Keyboard Z` | shift one gear down in manual transmission mode |
| `TRUCK_SHIFT_NEUTRAL` | `Keyboard D` | shift to neutral gear in manual transmission mode |
| `TRUCK_SHIFT_UP` | `Keyboard A` | shift one gear up in manual transmission mode |
| `TRUCK_SHIFT_GEAR_REVERSE` | — | shift directly to reverse gear |
| `TRUCK_SHIFT_GEAR1` | — | shift directly to first gear |
| `TRUCK_SHIFT_GEAR2` | — | shift directly to second gear |
| `TRUCK_SHIFT_GEAR3` | — | shift directly to third gear |
| `TRUCK_SHIFT_GEAR4` | — | shift directly to fourth gear |
| `TRUCK_SHIFT_GEAR5` | — | shift directly to 5th gear |
| `TRUCK_SHIFT_GEAR6` | — | shift directly to 6th gear |
| `TRUCK_SHIFT_GEAR7` | — | shift directly to 7th gear |
| `TRUCK_SHIFT_GEAR8` | — | shift directly to 8th gear |
| `TRUCK_SHIFT_GEAR9` | — | shift directly to 9th gear |
| `TRUCK_SHIFT_GEAR10` | — | shift directly to 10th gear |
| `TRUCK_SHIFT_GEAR11` | — | shift directly to 11th gear |
| `TRUCK_SHIFT_GEAR12` | — | shift directly to 12th gear |
| `TRUCK_SHIFT_GEAR13` | — | shift directly to 13th gear |
| `TRUCK_SHIFT_GEAR14` | — | shift directly to 14th gear |
| `TRUCK_SHIFT_GEAR15` | — | shift directly to 15th gear |
| `TRUCK_SHIFT_GEAR16` | — | shift directly to 16th gear |
| `TRUCK_SHIFT_GEAR17` | — | shift directly to 17th gear |
| `TRUCK_SHIFT_GEAR18` | — | shift directly to 18th gear |
| `TRUCK_SHIFT_LOWRANGE` | — | sets low range (1-6) for H-shaft |
| `TRUCK_SHIFT_MIDRANGE` | — | sets middle range (7-12) for H-shaft |
| `TRUCK_SHIFT_HIGHRANGE` | — | sets high range (13-18) for H-shaft |
| `TRUCK_CYCLE_GEAR_RANGES` | — | cycle through gear ranges |
| `TRUCK_SWITCH_SHIFT_MODES` | `Keyboard Q` | toggle between transmission modes |
| `AIRPLANE_STEER_RIGHT` | `Keyboard RIGHT` | steer right |
| `AIRPLANE_BRAKE` | `Keyboard B` | normal brake for an aircraft |
| `AIRPLANE_ELEVATOR_DOWN` | `Keyboard DOWN` | pull the elevator down in an aircraft. |
| `AIRPLANE_ELEVATOR_UP` | `Keyboard UP` | pull the elevator up in an aircraft. |
| `AIRPLANE_FLAPS_FULL` | `Keyboard CTRL+2` | full flaps in an aircraft. |
| `AIRPLANE_FLAPS_LESS` | `Keyboard EXPL+1` | one step less flaps. |
| `AIRPLANE_FLAPS_MORE` | `Keyboard EXPL+2` | one step more flaps. |
| `AIRPLANE_FLAPS_NONE` | `Keyboard CTRL+1` | no flaps. |
| `AIRPLANE_PARKING_BRAKE` | `Keyboard P` | airplane parking brake. |
| `AIRPLANE_REVERSE` | `Keyboard R` | reverse the turboprops |
| `AIRPLANE_RUDDER_LEFT` | `Keyboard Z` | rudder left |
| `AIRPLANE_RUDDER_RIGHT` | `Keyboard X` | rudder right |
| `AIRPLANE_STEER_LEFT` | `Keyboard LEFT` | steer left |
| `AIRPLANE_STEER_RIGHT` | `Keyboard RIGHT` | steer right |
| `AIRPLANE_THROTTLE_AXIS` | `None` | throttle axis. Only use this if you have fitting hardware :) (i.e. a Slider) |
| `AIRPLANE_THROTTLE_DOWN` | `Keyboard EXPL+PGDOWN` | decreases the airplane thrust |
| `AIRPLANE_THROTTLE_FULL` | `Keyboard CTRL+PGUP` | full thrust |
| `AIRPLANE_THROTTLE_NO` | `Keyboard CTRL+PGDOWN` | no thrust |
| `AIRPLANE_THROTTLE_UP` | `Keyboard EXPL+PGUP` | increase the airplane thrust |
| `AIRPLANE_TOGGLE_ENGINES` | `Keyboard CTRL+HOME` | switch all engines on / off |
| `AIRPLANE_AIRBRAKES_NONE` | `Keyboard CTRL+3` | no airbrakes |
| `AIRPLANE_AIRBRAKES_FULL` | `Keyboard CTRL+4` | full airbrakes |
| `AIRPLANE_AIRBRAKES_LESS` | `Keyboard EXPL+3` | less airbrakes |
| `AIRPLANE_AIRBRAKES_MORE` | `Keyboard EXPL+4` | more airbrakes |
| `AIRPLANE_THROTTLE` | — | airplane throttle |
| `BOAT_CENTER_RUDDER` | `Keyboard PGDOWN` | center the rudder |
| `BOAT_REVERSE` | `Keyboard PGUP` | no thrust |
| `BOAT_STEER_LEFT` | `Keyboard LEFT` | steer left a step |
| `BOAT_STEER_LEFT_AXIS` | `None` | steer left (analog value!) |
| `BOAT_STEER_RIGHT` | `Keyboard RIGHT` | steer right a step |
| `BOAT_STEER_RIGHT_AXIS` | `None` | steer right (analog value!) |
| `BOAT_THROTTLE_AXIS` | `None` | throttle axis. Only use this if you have fitting hardware :) (i.e. a Slider) |
| `BOAT_THROTTLE_DOWN` | `Keyboard DOWN` | decrease throttle |
| `BOAT_THROTTLE_UP` | `Keyboard UP` | increase throttle |
| `COMMANDS_01` | `Keyboard EXPL+F1` | Command 1 |
| `COMMANDS_02` | `Keyboard EXPL+F2` | Command 2 |
| `COMMANDS_03` | `Keyboard EXPL+F3` | Command 3 |
| `COMMANDS_04` | `Keyboard EXPL+F4` | Command 4 |
| `COMMANDS_05` | `Keyboard EXPL+F5` | Command 5 |
| `COMMANDS_06` | `Keyboard EXPL+F6` | Command 6 |
| `COMMANDS_07` | `Keyboard EXPL+F7` | Command 7 |
| `COMMANDS_08` | `Keyboard EXPL+F8` | Command 8 |
| `COMMANDS_09` | `Keyboard EXPL+F9` | Command 9 |
| `COMMANDS_10` | `Keyboard EXPL+F10` | Command 10 |
| `COMMANDS_11` | `Keyboard EXPL+F11` | Command 11 |
| `COMMANDS_12` | `Keyboard EXPL+F12` | Command 12 |
| `COMMANDS_13` | `Keyboard EXPL+CTRL+F1` | Command 13 |
| `COMMANDS_14` | `Keyboard EXPL+CTRL+F2` | Command 14 |
| `COMMANDS_15` | `Keyboard EXPL+CTRL+F3` | Command 15 |
| `COMMANDS_16` | `Keyboard EXPL+CTRL+F4` | Command 16 |
| `COMMANDS_17` | `Keyboard EXPL+CTRL+F5` | Command 17 |
| `COMMANDS_18` | `Keyboard EXPL+CTRL+F6` | Command 18 |
| `COMMANDS_19` | `Keyboard EXPL+CTRL+F7` | Command 19 |
| `COMMANDS_20` | `Keyboard EXPL+CTRL+F8` | Command 20 |
| `COMMANDS_21` | `Keyboard EXPL+CTRL+F9` | Command 21 |
| `COMMANDS_22` | `Keyboard EXPL+CTRL+F10` | Command 22 |
| `COMMANDS_23` | `Keyboard EXPL+CTRL+F11` | Command 23 |
| `COMMANDS_24` | `Keyboard EXPL+CTRL+F12` | Command 24 |
| `COMMANDS_25` | `Keyboard EXPL+SHIFT+F1` | Command 25 |
| `COMMANDS_26` | `Keyboard EXPL+SHIFT+F2` | Command 26 |
| `COMMANDS_27` | `Keyboard EXPL+SHIFT+F3` | Command 27 |
| `COMMANDS_28` | `Keyboard EXPL+SHIFT+F4` | Command 28 |
| `COMMANDS_29` | `Keyboard EXPL+SHIFT+F5` | Command 29 |
| `COMMANDS_30` | `Keyboard EXPL+SHIFT+F6` | Command 30 |
| `COMMANDS_31` | `Keyboard EXPL+SHIFT+F7` | Command 31 |
| `COMMANDS_32` | `Keyboard EXPL+SHIFT+F8` | Command 32 |
| `COMMANDS_33` | `Keyboard EXPL+SHIFT+F9` | Command 33 |
| `COMMANDS_34` | `Keyboard EXPL+SHIFT+F10` | Command 34 |
| `COMMANDS_35` | `Keyboard EXPL+SHIFT+F11` | Command 35 |
| `COMMANDS_36` | `Keyboard EXPL+SHIFT+F12` | Command 36 |
| `COMMANDS_37` | `Keyboard EXPL+ALT+F1` | Command 37 |
| `COMMANDS_38` | `Keyboard EXPL+ALT+F2` | Command 38 |
| `COMMANDS_39` | `Keyboard EXPL+ALT+F3` | Command 39 |
| `COMMANDS_40` | `Keyboard EXPL+ALT+F4` | Command 40 |
| `COMMANDS_41` | `Keyboard EXPL+ALT+F5` | Command 41 |
| `COMMANDS_42` | `Keyboard EXPL+ALT+F6` | Command 42 |
| `COMMANDS_43` | `Keyboard EXPL+ALT+F7` | Command 43 |
| `COMMANDS_44` | `Keyboard EXPL+ALT+F8` | Command 44 |
| `COMMANDS_45` | `Keyboard EXPL+ALT+F9` | Command 45 |
| `COMMANDS_46` | `Keyboard EXPL+ALT+F10` | Command 46 |
| `COMMANDS_47` | `Keyboard EXPL+ALT+F11` | Command 47 |
| `COMMANDS_48` | `Keyboard EXPL+ALT+F12` | Command 48 |
| `COMMANDS_49` | `Keyboard EXPL+CTRL+SHIFT+F1` | Command 49 |
| `COMMANDS_50` | `Keyboard EXPL+CTRL+SHIFT+F2` | Command 50 |
| `COMMANDS_51` | `Keyboard EXPL+CTRL+SHIFT+F3` | Command 51 |
| `COMMANDS_52` | `Keyboard EXPL+CTRL+SHIFT+F4` | Command 52 |
| `COMMANDS_53` | `Keyboard EXPL+CTRL+SHIFT+F5` | Command 53 |
| `COMMANDS_54` | `Keyboard EXPL+CTRL+SHIFT+F6` | Command 54 |
| `COMMANDS_55` | `Keyboard EXPL+CTRL+SHIFT+F7` | Command 55 |
| `COMMANDS_56` | `Keyboard EXPL+CTRL+SHIFT+F8` | Command 56 |
| `COMMANDS_57` | `Keyboard EXPL+CTRL+SHIFT+F9` | Command 57 |
| `COMMANDS_58` | `Keyboard EXPL+CTRL+SHIFT+F10` | Command 58 |
| `COMMANDS_59` | `Keyboard EXPL+CTRL+SHIFT+F11` | Command 59 |
| `COMMANDS_60` | `Keyboard EXPL+CTRL+SHIFT+F12` | Command 60 |
| `COMMANDS_61` | `Keyboard EXPL+CTRL+ALT+F1` | Command 61 |
| `COMMANDS_62` | `Keyboard EXPL+CTRL+ALT+F2` | Command 62 |
| `COMMANDS_63` | `Keyboard EXPL+CTRL+ALT+F3` | Command 63 |
| `COMMANDS_64` | `Keyboard EXPL+CTRL+ALT+F4` | Command 64 |
| `COMMANDS_65` | `Keyboard EXPL+CTRL+ALT+F5` | Command 65 |
| `COMMANDS_66` | `Keyboard EXPL+CTRL+ALT+F6` | Command 66 |
| `COMMANDS_67` | `Keyboard EXPL+CTRL+ALT+F7` | Command 67 |
| `COMMANDS_68` | `Keyboard EXPL+CTRL+ALT+F8` | Command 68 |
| `COMMANDS_69` | `Keyboard EXPL+CTRL+ALT+F9` | Command 69 |
| `COMMANDS_70` | `Keyboard EXPL+CTRL+ALT+F10` | Command 70 |
| `COMMANDS_71` | `Keyboard EXPL+CTRL+ALT+F11` | Command 71 |
| `COMMANDS_72` | `Keyboard EXPL+CTRL+ALT+F12` | Command 72 |
| `COMMANDS_73` | `Keyboard EXPL+CTRL+SHIFT+ALT+F1` | Command 73 |
| `COMMANDS_74` | `Keyboard EXPL+CTRL+SHIFT+ALT+F2` | Command 74 |
| `COMMANDS_75` | `Keyboard EXPL+CTRL+SHIFT+ALT+F3` | Command 75 |
| `COMMANDS_76` | `Keyboard EXPL+CTRL+SHIFT+ALT+F4` | Command 76 |
| `COMMANDS_77` | `Keyboard EXPL+CTRL+SHIFT+ALT+F5` | Command 77 |
| `COMMANDS_78` | `Keyboard EXPL+CTRL+SHIFT+ALT+F6` | Command 78 |
| `COMMANDS_79` | `Keyboard EXPL+CTRL+SHIFT+ALT+F7` | Command 79 |
| `COMMANDS_80` | `Keyboard EXPL+CTRL+SHIFT+ALT+F8` | Command 80 |
| `COMMANDS_81` | `Keyboard EXPL+CTRL+SHIFT+ALT+F9` | Command 81 |
| `COMMANDS_82` | `Keyboard EXPL+CTRL+SHIFT+ALT+F10` | Command 82 |
| `COMMANDS_83` | `Keyboard EXPL+CTRL+SHIFT+ALT+F11` | Command 83 |
| `COMMANDS_84` | `Keyboard EXPL+CTRL+SHIFT+ALT+F12` | Command 84 |
| `CHARACTER_BACKWARDS` | `Keyboard S` | step backwards with the character |
| `CHARACTER_FORWARD` | `Keyboard W` | step forward with the character |
| `CHARACTER_JUMP` | `Keyboard SPACE` | let the character jump |
| `CHARACTER_LEFT` | `Keyboard LEFT` | rotate character left |
| `CHARACTER_RIGHT` | `Keyboard RIGHT` | rotate character right |
| `CHARACTER_RUN` | `Keyboard SHIFT+W` | let the character run |
| `CHARACTER_SIDESTEP_LEFT` | `Keyboard A` | sidestep to the left |
| `CHARACTER_SIDESTEP_RIGHT` | `Keyboard D` | sidestep to the right |
| `CHARACTER_ROT_UP` | `Keyboard UP` | rotate view up |
| `CHARACTER_ROT_DOWN` | `Keyboard DOWN` | rotate view down |
| `CAMERA_CHANGE` | `Keyboard EXPL+C` | change camera mode |
| `CAMERA_LOOKBACK` | `Keyboard NUMPAD1` | look back (toggles between normal and lookback) |
| `CAMERA_RESET` | `Keyboard NUMPAD5` | reset the camera position |
| `CAMERA_ROTATE_DOWN` | `Keyboard NUMPAD2` | rotate camera down |
| `CAMERA_ROTATE_LEFT` | `Keyboard NUMPAD4` | rotate camera left |
| `CAMERA_ROTATE_RIGHT` | `Keyboard NUMPAD6` | rotate camera right |
| `CAMERA_ROTATE_UP` | `Keyboard NUMPAD8` | rotate camera up |
| `CAMERA_ZOOM_IN` | `Keyboard EXPL+NUMPAD9` | zoom camera in |
| `CAMERA_ZOOM_IN_FAST` | `Keyboard SHIFT+NUMPAD9` | zoom camera in faster |
| `CAMERA_ZOOM_OUT` | `Keyboard EXPL+NUMPAD3` | zoom camera out |
| `CAMERA_ZOOM_OUT_FAST` | `Keyboard SHIFT+NUMPAD3` | zoom camera out faster |
| `CAMERA_FREE_MODE_FIX` | `Keyboard EXPL+ALT+C` | fix the camera to a position |
| `CAMERA_FREE_MODE` | `Keyboard EXPL+SHIFT+C` | enable / disable free camera mode |
| `CAMERA_UP` | `Keyboard Q` | move camera up |
| `CAMERA_DOWN` | `Keyboard Z` | move camera down |
| `SKY_DECREASE_TIME` | `Keyboard EXPL+SUBTRACT` | decrease day-time |
| `SKY_DECREASE_TIME_FAST` | `Keyboard SHIFT+SUBTRACT` | decrease day-time a lot faster |
| `SKY_INCREASE_TIME` | `Keyboard EXPL+ADD` | increase day-time |
| `SKY_INCREASE_TIME_FAST` | `Keyboard SHIFT+ADD` | increase day-time a lot faster |
| `GRASS_LESS` | — | EXPERIMENTAL: remove some grass |
| `GRASS_MORE` | — | EXPERIMENTAL: add some grass |
| `GRASS_MOST` | — | EXPERIMENTAL: set maximum amount of grass |
| `GRASS_NONE` | — | EXPERIMENTAL: remove grass completely |
| `GRASS_SAVE` | — | EXPERIMENTAL: save changes to the grass density image |
| `SURVEY_MAP_TOGGLE_ICONS` | `Keyboard EXPL+CTRL+SHIFT+ALT+TAB` | toggle map icons |
| `SURVEY_MAP_TOGGLE` | `Keyboard EXPL+CTRL+SHIFT+TAB` | toggle map |
| `SURVEY_MAP_CYCLE` | `Keyboard EXPL+TAB` | cycle map modes |
| `SURVEY_MAP_ZOOM_IN` | `Keyboard EXPL+CTRL+TAB` | zoom in |
| `SURVEY_MAP_ZOOM_OUT` | `Keyboard EXPL+SHIFT+TAB` | zoom out |
| `MENU_DOWN` | `Keyboard DOWN` | select next element in current category |
| `MENU_LEFT` | `Keyboard LEFT` | select previous category |
| `MENU_RIGHT` | `Keyboard RIGHT` | select next category |
| `MENU_SELECT` | `Keyboard EXPL+RETURN` | select focussed item and close menu |
| `MENU_UP` | `Keyboard UP` | select previous element in current category |
| `TRUCKEDIT_RELOAD` | `Keyboard EXPL+SHIFT+CTRL+R` | reload truck |
| `ROAD_EDITOR_POINT_INSERT` | `Keyboard EXPL+INSERT` | insert road point |
| `ROAD_EDITOR_POINT_GOTO` | `Keyboard EXPL+G` | go to road point |
| `ROAD_EDITOR_POINT_SET_POS` | `Keyboard EXPL+M` | set road point position |
| `ROAD_EDITOR_POINT_DELETE` | `Keyboard EXPL+DELETE` | delete road point |
| `ROAD_EDITOR_REBUILD_MESH` | `Keyboard EXPL+B` | regenerate road mesh |

Key names accepted after the modifiers: `0`, `1`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, `9`, `A`, `ABNT_C1`, `ABNT_C2`, `ADD`, `APOSTROPHE`, `APPS`, `AT`, `AX`, `B`, `BACK`, `BACKSLASH`, `C`, `CALCULATOR`, `CAPITAL`, `COLON`, `COMMA`, `CONVERT`, `D`, `DECIMAL`, `DELETE`, `DIVIDE`, `DOWN`, `E`, `END`, `EQUALS`, `ESCAPE`, `F`, `F1`, `F10`, `F11`, `F12`, `F13`, `F14`, `F15`, `F2`, `F3`, `F4`, `F5`, `F6`, `F7`, `F8`, `F9`, `G`, `GRAVE`, `H`, `HOME`, `I`, `INSERT`, `J`, `K`, `KANA`, `KANJI`, `L`, `LBRACKET`, `LCONTROL`, `LEFT`, `LMENU`, `LSHIFT`, `LWIN`, `M`, `MAIL`, `MEDIASELECT`, `MEDIASTOP`, `MINUS`, `MULTIPLY`, `MUTE`, `MYCOMPUTER`, `N`, `NEXTTRACK`, `NOCONVERT`, `NUMLOCK`, `NUMPAD0`, `NUMPAD1`, `NUMPAD2`, `NUMPAD3`, `NUMPAD4`, `NUMPAD5`, `NUMPAD6`, `NUMPAD7`, `NUMPAD8`, `NUMPAD9`, `NUMPADCOMMA`, `NUMPADENTER`, `NUMPADEQUALS`, `O`, `OEM_102`, `P`, `PAUSE`, `PERIOD`, `PGDOWN`, `PGUP`, `PLAYPAUSE`, `POWER`, `PREVTRACK`, `Q`, `R`, `RBRACKET`, `RCONTROL`, `RETURN`, `RIGHT`, `RMENU`, `RSHIFT`, `RWIN`, `S`, `SCROLL`, `SEMICOLON`, `SLASH`, `SLEEP`, `SPACE`, `STOP`, `SUBTRACT`, `SYSRQ`, `T`, `TAB`, `U`, `UNDERLINE`, `UNLABELED`, `UP`, `V`, `VOLUMEDOWN`, `VOLUMEUP`, `W`, `WAKE`, `WEBBACK`, `WEBFAVORITES`, `WEBFORWARD`, `WEBHOME`, `WEBREFRESH`, `WEBSEARCH`, `WEBSTOP`, `X`, `Y`, `YEN`, `Z`.
