# source/main/resources/rig_def_fileformat/RigDef_Regexes.h

> The keyword recogniser and the `axles` item pattern.

**Needs** — nothing
**Used by** — [`RigDef_Parser.cpp`](RigDef_Parser.cpp.md)
**Tier floor** — T4

## Purpose

Two patterns. A rebuild can use any matching technique; what must survive is the keyword list, its order (which defines keyword ids) and the block-vs-inline matching rule.

## State

Stateless (compiled patterns).

## Keyword recogniser

**Contract** — case-insensitive; the list is alphabetical and position `k` (1-based) corresponds to `Keyword` value `k` in [`Application.h`](../../Application.h.md#rigdefkeyword). Each entry is either:

- **block** — `^name[blank]*$` (keyword alone on the line), or
- **inline** — `^name[separators]+.*$` (keyword followed by arguments), or
- **inline-unseparated** — `^name.*$` (only `forset`).

Inline keywords: `add_animation`, `AntiLockBrakes`, `author`, `cruisecontrol`, `default_skin`, `detacher_group`, `extcamera`, `fileformatversion`, `fileinfo`, `flexbody_camera_mode`, `forvert`, `guid`, `prop_camera_mode`, `section`, `sectionconfig`, `set_beam_defaults`, `set_beam_defaults_scale`, `set_collision_range`, `set_default_minimass`, `set_inertia_defaults`, `set_managedmaterials_options`, `set_node_defaults`, `set_skeleton_settings`, `SlopeBrake`, `speedlimiter`, `submesh_groundmodel`, `TractionControl`; unseparated: `forset`. Everything else in the list is a block keyword: `airbrakes`, `animators`, `assetpacks`, `axles`, `backmesh`, `beams`, `brakes`, `cab`, `camerarail`, `cameras`, `cinecam`, `collisionboxes`, `commands`, `commands2`, `comment`, `contacters`, `customdashboardinputs`, `description`, `disabledefaultsounds`, `enable_advanced_deformation`, `end`, `end_comment`, `end_description`, `end_section`, `engine`, `engoption`, `engturbo`, `envmap`, `exhausts`, `fixes`, `flares`, `flares2`, `flares3`, `flaregroups_no_import`, `flexbodies`, `flexbodywheels`, `forwardcommands`, `fusedrag`, `globals`, `guisettings`, `help`, `hideInChooser`, `hookgroup`, `hooks`, `hydros`, `importcommands`, `interaxles`, `lockgroups`, `lockgroup_default_nolock`, `managedmaterials`, `materialflarebindings`, `meshwheels`, `meshwheels2`, `minimass`, `nodecollision`, `nodes`, `nodes2`, `particles`, `pistonprops`, `props`, `railgroups`, `rescuer`, `rigidifiers`, `rollon`, `ropables`, `ropes`, `rotators`, `rotators2`, `screwprops`, `scripts`, `set_shadows`, `shocks`, `shocks2`, `shocks3`, `slidenode_connect_instantly`, `slidenodes`, `soundsources`, `soundsources2`, `submesh`, `texcoords`, `ties`, `torquecurve`, `transfercase`, `triggers`, `turbojets`, `turboprops`, `turboprops2`, `videocamera`, `wheeldetachers`, `wheels`, `wheels2`, `wings`.

Separators: space, tab, `,`, `:`, `|`.

**Notes** — the recogniser's list and the `Keyword` enum are maintained by hand in lock-step (120 entries; `sectionconfig`, `set_shadows` and `SlopeBrake` are recognised and have enum members but no parser). Adding a keyword in one place and not the other silently shifts every later id; a rebuild should generate both from one table.

## Axles item pattern

**Contract** — one comma-separated part of an `axles` line is `w1(<node> <node>)`, `w2(<node> <node>)` or `d(<letters o/l/s/v>)`, with optional surrounding blanks and an optional trailing `;…` or `//…` comment. Node ids here are `[A-Za-z0-9_-]+` separated by blanks.
