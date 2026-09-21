# source/main/gui/DashBoardManager.cpp

> Dashboard mod resolution, layout attribute parsing, and per-frame widget animation.

**Needs** — [`DashBoardManager.h`](DashBoardManager.h.md) · [`physics/Actor.h`](../physics/Actor.h.md) · [`Application.h`](../Application.h.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`system/Console.h`](../system/Console.h.md) · [`utils/GenericFileFormat.h`](../utils/GenericFileFormat.h.md) · [`GUIManager.h`](GUIManager.h.md) · [`RTTLayer.h`](RTTLayer.h.md) · [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md) · [`utils/Utils.h`](../utils/Utils.h.md) · [Seam: Retained layout GUI](../../../SYSTEM-REQUIREMENTS.md#seam-retained-layout-gui)
**Used by** — callers of [`DashBoardManager.h`](DashBoardManager.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`DashBoardManager.h`](DashBoardManager.h.md).

## State

See header.

## Construction

Create the built-in slots with their names and types (enabled). Custom inputs are appended.

## `registerCustomInput(name, type)`

Rejected (warning) if the name already exists or the type is invalid; otherwise appended with id `MAX + n`.

## `loadDashBoard(file, flags, rtt layer)`

```text
no HUD/RTT/RENDERDASH flag → nothing
".dashboard" file (a dashboard mod in the content cache):
  load its bundle; pick a layout (below); read its custom inputs; load every "<base>*.resource" GUI resource file;
  if "<base>.as" exists, request it be loaded as an ACTOR script for this vehicle
otherwise the name is the layout itself
no layout → warning, none
RTT_TEXTURE: new dashboard into the texture layer, hidden; unless STACKABLE, mark RTT loaded
RENDERDASH:  new dashboard into the texture layer, visible
SCREEN_HUD:  new dashboard on screen, visible; unless STACKABLE, mark HUD loaded
guisettings from the vehicle: tacho/speedo/help material → first texture name into the guisetting_*_tex slots;
  if "use engine max RPM": every tacho_rpm animation's vmax = shift-up RPM;
  if a non-default speedo max: every speedo_kph animation's vmax = that value
```

**Notes** — the guisettings rescaling applies to the last dashboard created by this call only.

### Choosing a layout inside a dashboard mod

Candidates are `<base>*.layout`. Boats and aircraft take the first. Trucks (or RENDERDASH) choose by tags in the file name: `<N>rpm` (tacho range) and `kph`/`mph`. Among layouts whose unit matches the imperial/metric setting, take the one whose RPM is the smallest ≥ the vehicle's shift-up RPM; failing that the largest below it (with a warning). If no layout has the desired unit, repeat ignoring units (warning). Finally, any layout.

### Custom inputs from the `.dashboard` file

Generic-document lines `dashboard_custom_input <name> <bool|float|int|string>` register inputs; an unknown type is an error.

## Layout parsing — which widgets are live

Widgets are read recursively. A widget with user attribute `debug`, or named `DEBUG`, is hidden and skipped with its children; `_Main` becomes the root sized to the screen or texture. Otherwise:

```text
IF user attribute "anim" present: for n = 1, 2, …: read anim/link/min/max/vmin/vmax/texture/format/direction
     (suffix "" for n=1, else n) while anim<n> is non-empty
  link = "<slot name>" | "<slot name> > x" | "<slot name> < x"  (XML-escaped &gt; &lt; accepted); unknown slot → skip widget
  first link = visibility slot
  graphical kinds (one per widget; a second one skips the widget): series, textcolor/textcolour, textformat, textstring,
     lamp, imagetexture — needs an image box (series, lamp, imagetexture) or text box (the text kinds)
  geometric kinds (≤10): rotate (needs a rotating skin; pivot = widget centre), scale and translate (need direction)
  unknown anim or bad direction → skip widget
ELSE IF "link" present: a visibility-only control
recurse into children
```

A skipped widget is simply not animated; loading continues.

## `DashBoard::update` — per frame

For each control:

- **lamp** — state = value > x, value < x, or value > 0; on change, texture `<texture>-on.png` / `-off.png`.
- **series** — texture `<texture>-<int(value)>.png`.
- **textformat** — value formatted with the printf-style format (default: shortest decimal); if the result equals the format applied to −0, show the format of 0 instead (no "-0.0").
- **textstring** — caption = text slot. **imagetexture** — texture = text slot when non-empty.
- **rotate** — angle = linear map of value from [vmin, vmax] to [min, max] degrees, clamped.
- **scale** — amount = same linear map (unclamped); grow the widget by it toward the direction (up/left also move the origin).
- **translate** — offsets summed over all translate animations per axis; the widget is then placed at initial position + offsets (every frame, for every control).

Float inputs pass through `getSmoothNumeric`: `0.98 × previous raw + 0.02 × current raw`.

**Notes** — because the "previous" stored is the raw sample, not the output, this is a one-frame lag, not exponential smoothing; a rebuild that wants smoothing stores the output. `textcolor` is parsed but has no effect. `updateFeatures` shows/hides each control from its visibility slot's `enabled` flag.

## Visibility and resize

Texture dashboards toggle without fading. `windowResized` sizes the root to the screen or to its texture. Destroying a dashboard unloads its layout and resources and returns its render-to-texture layer to the pool.
