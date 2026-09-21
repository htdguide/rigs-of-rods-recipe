# source/main/resources/rig_def_fileformat/RigDef_File.h

> The in-memory document of a truck file: one record type per section line, grouped into modules.

**Needs** — [`Application.h`](../../Application.h.md) (`Keyword`, format-stable enums) · [`utils/BitFlags.h`](../../utils/BitFlags.h.md) · [`RigDef_Node.h`](RigDef_Node.h.md) · [`physics/SimConstants.h`](../../physics/SimConstants.h.md)
**Used by** — [`physics/ActorExport.cpp`](../../physics/ActorExport.cpp.md) · [`physics/air/TurboJet.h`](../../physics/air/TurboJet.h.md) · [`physics/flex/FlexBody.cpp`](../../physics/flex/FlexBody.cpp.md) · [`physics/flex/FlexBody.h`](../../physics/flex/FlexBody.h.md) · [`physics/flex/FlexFactory.cpp`](../../physics/flex/FlexFactory.cpp.md) · [`resources/CacheSystem.h`](../CacheSystem.h.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.h`](../addonpart_fileformat/AddonPartFileFormat.h.md) · [`RigDef_File.cpp`](RigDef_File.cpp.md) · [`RigDef_Parser.cpp`](RigDef_Parser.cpp.md) · [`RigDef_Parser.h`](RigDef_Parser.h.md) · [`RigDef_SequentialImporter.h`](RigDef_SequentialImporter.h.md) · [`RigDef_Serializer.cpp`](RigDef_Serializer.cpp.md) · [`RigDef_Serializer.h`](RigDef_Serializer.h.md) · [`RigDef_Validator.h`](RigDef_Validator.h.md)
**Tier floor** — T4

## Purpose

The parser fills this structure *without interpreting it*; the [actor spawner](../../physics/ActorSpawner.cpp.md) later turns it into a simulated vehicle. Keeping a faithful document (rather than spawning while parsing) is what allows validation before spawn, add-on parts and tune-ups to be merged in, and the file to be written back out. Design rules the original follows and a rebuild should keep:

- one record type per kind of line; the record holds only that line's arguments, in file order (helper data prefixed `_`);
- every section is a list named after its keyword;
- option letters become bit sets (or an ordered list when order matters, e.g. differential types);
- `set_*_defaults` directives are **shared snapshots**: each affected line keeps a reference to the defaults object that was current when it was read, so later directives do not rewrite earlier lines.

Line syntax for every record is in [`RigDef_Parser.cpp`](RigDef_Parser.cpp.md#section-syntax); defaults are listed here.

## State

```text
RECORD Document
  name                          : text      # first non-comment line of the file
  hash                          : text      # SHA-1 of the file, filled by the loader
  # file-wide flags (keywords with no arguments):
  hide_in_chooser, enable_advanced_deformation, slide_nodes_connect_instantly,
  rollon, forward_commands, import_commands, lockgroup_default_nolock,
  rescuer, disable_default_sounds : bool = false
  root_module  : Module named "_Root_"      # always exists
  user_modules : map<name, Module>          # created by 'section <n> <name>'

RECORD Module
  name             : text
  origin_addonpart : optional cache entry   # set when the module came from an .addonpart
  <one list per section>                    # see table below
  _comments        : list<DocComment>       # comment blocks attached to the following element
  _hint_*_linenumber : int                  # editor hints (nodes/beams block extents)
```

A **module** is a named optional part of the vehicle (the file calls them `section`s and the UI calls them configurations). The spawn process uses the root module plus the modules the player selected.

### Shared helper records

```text
RECORD BeamDefaults            # from 'set_beam_defaults'; parser seeds game defaults
  springiness          = DEFAULT_SPRING (9,000,000)
  damping_constant     = DEFAULT_DAMP (12,000)
  deformation_threshold= BEAM_DEFORM (400,000)
  breaking_threshold   = BEAM_BREAK (1,000,000)
  visual_beam_diameter = DEFAULT_BEAM_DIAMETER (0.05 m)
  beam_material_name   = "tracks/beam"
  plastic_deform_coef  = 0
  scale : { springiness, damping, deform, break } all 1.0   # from 'set_beam_defaults_scale'
  _enable_advanced_deformation : bool  # was the directive seen before this line?
  _is_user_defined, _is_plastic_deform_coef_user_defined : bool
  scaled value = value * scale.<same>

RECORD NodeDefaults            # from 'set_node_defaults'
  load_weight = -1 (use game default)   friction = 1   volume = 1   surface = 1   options = 0

RECORD DefaultMinimass         # from 'set_default_minimass'
  min_mass_kg = DEFAULT_MINIMASS (50)

RECORD Inertia                 # from 'set_inertia_defaults' or inline on commands/hydros/rotators
  start_delay_factor = 0, stop_delay_factor = 0, start_function = "", stop_function = ""

RECORD DocComment
  text : text                  # original comment lines, each keeping its ';' or '//'
  keyword, index               # the element it precedes
```

### Section records and defaults

| List | Record fields (defaults) |
|---|---|
| `nodes` (also `nodes2`) | see [`RigDef_Node.h`](RigDef_Node.h.md) |
| `beams` | nodes[2]; options {i invisible, r rope, s support}; extension_break_limit (support beams only); detacher_group; defaults |
| `shocks` | nodes[2], spring_rate, damping, short_bound, long_bound, precompression (1.0); options {i, L active-left, R active-right, m metric}; detacher_group; beam defaults |
| `shocks2` | nodes[2], spring_in, damp_in, progress_factor_spring_in, progress_factor_damp_in, spring_out, damp_out, progress_factor_spring_out, progress_factor_damp_out, short_bound, long_bound, precompression; options {i, s soft bump bounds, m metric, M absolute metric} |
| `shocks3` | nodes[2], spring_in, damp_in, damp_in_slow, split_vel_in, damp_in_fast, spring_out, damp_out, damp_out_slow, split_vel_out, damp_out_fast, short_bound, long_bound, precompression; options {i, m, M} |
| `hydros` | nodes[2], lengthening_factor; options (bit set, see parser); inertia; inertia defaults; beam defaults; detacher_group |
| `commands2` (also `commands`) | nodes[2], shorten_rate, lengthen_rate, max_contraction, max_extension, contract_key, extend_key, description, inertia, affect_engine (1.0), needs_engine (true), plays_sound (true), flags i/r/c/f/p/o, beam defaults, inertia defaults, detacher_group |
| `triggers` | nodes[2], contraction_trigger_limit, expansion_trigger_limit, shortbound_trigger_action, longbound_trigger_action, options, boundary_timer (1.0 s), beam defaults, detacher_group |
| `animators` | nodes[2], lengthening_factor, flags (sources + vis/inv + short/long limit), short_limit, long_limit, aero_animator {flags, engine index}, inertia & beam defaults, detacher_group |
| `ties` | root_node, max_reach_length, auto_shorten_rate, min_length, max_length, options {i, s no-self-lock}, max_stress (100,000), group (−1), beam defaults, detacher_group |
| `ropes` | root_node, end_node, invisible (false), beam defaults, detacher_group |
| `ropables` | node, group (−1), has_multilock (false) |
| `hooks` | node, hook_range (0.4), speed_coef (1.0), max_force (10,000,000), hookgroup (−1), lockgroup (−1 = default), timer (5 s), min_range (0); flags self_lock, auto_lock, no_disable, no_rope, visible |
| `lockgroups` | number, nodes[] (special numbers: −1 default, 9999 no-lock) |
| `slidenodes` | slide_node, rail ranges[], optional spring_rate, break_force, tolerance, attachment_rate, railgroup_id, max_attach_dist (each with a "was set" flag); constraint flags {attach all/foreign/self/none} |
| `railgroups` | id, node ranges[] |
| `fixes`, `contacters` | node references |
| `cameras` | center, back, left nodes |
| `cinecam` | position, nodes[8], spring (8,000), damping (800), node_mass (20 kg), node & beam defaults |
| `camerarail` | nodes[] (one rail per block) |
| `collisionboxes` | nodes[] |
| `wheels` | radius, width, num_rays, nodes[2], rigidity_node, braking, propulsion, reference_arm_node, mass, springiness, damping, face_material ("tracks/wheelface"), band_material ("tracks/wheelband1"), node & beam defaults |
| `wheels2` | rim_radius, tyre_radius, width, num_rays, nodes[2], rigidity_node, braking, propulsion, arm node, mass, rim_springiness, rim_damping, tyre_springiness, tyre_damping, face & band material |
| `meshwheels`, `meshwheels2` | tyre_radius, rim_radius, width, num_rays, nodes[2], rigidity_node, braking, propulsion, arm node, mass, spring, damping, side (l/r), mesh_name, material_name |
| `flexbodywheels` | tyre_radius, rim_radius, width, num_rays, nodes[2], rigidity_node, braking, propulsion, arm node, mass, tyre_springiness, tyre_damping, rim_springiness, rim_damping, side, rim_mesh_name, tyre_mesh_name |
| `wheeldetachers` | wheel_id, detacher_group |
| `engine` | shift_down_rpm, shift_up_rpm, torque, global_gear_ratio, reverse_gear_ratio, neutral_gear_ratio, forward gear_ratios[] |
| `engoption` | inertia (10), type (c car / e electric / t truck), clutch_force, shift_time, clutch_time, post_shift_time, idle_rpm, stall_rpm, max_idle_mixture, min_idle_mixture, braking_torque (all −1 = use engine default) |
| `engturbo` | version, inertia factor (1), n_turbos (1, max 4), param1..param11 (9999 = default) |
| `torquecurve` | samples[(rpm fraction, torque %)] or a predefined curve name |
| `brakes` | default_braking_force (30,000), parking_brake_force (−1 = derive) |
| `antilockbrakes` | regulation_force, min_speed, pulse_per_sec, is_on (true), no_dashboard, no_toggle |
| `tractioncontrol` | regulation_force, wheel_slip, fade_speed, pulse_per_sec, is_on (false), no_dashboard, no_toggle |
| `cruisecontrol` | min_speed, autobrake |
| `speedlimiter` | max_speed, enabled |
| `axles` | wheels[2][2] (two wheels × two axis nodes), differential types (ordered list of o/l/s/v) |
| `interaxles` | a1, a2 (0-based axle indices), differential types |
| `transfercase` | a1 (0-based), a2 (−1 = none), has_2wd (true), has_2wd_lo (false), gear_ratios ([1.0] + extra) |
| `globals` | dry_mass, cargo_mass, material_name |
| `minimass` | global_min_mass_kg, option (n / l skip nodes with load weight) |
| `set_collision_range` | node_collision_range (−1 = default) |
| `set_skeleton_settings` | visibility_range (150 m), beam_thickness (0.01 m) |
| `props` | ref/x/y nodes, offset, rotation (degrees), mesh_name, animations[], camera mode (always visible), special kind + beacon {flare material "tracks/beaconflare", colour (1, 0.5, 0)} + dashboard {mesh "dirwheel.mesh", offset, rotation angle 160°} |
| `flexbodies` | ref/x/y nodes, offset, rotation, mesh_name, animations[], forset ranges → node list, forvert bindings, camera mode |
| `flares2` (also `flares`), `flares3` | ref/x/y nodes, offset (0, 0, 1), type ('f'), control_number (−1; user lights), dashboard_link (dashboard lights), blink_delay_ms (−2 = default), size (−1 = default), material; flares3 also inertia defaults |
| `flaregroups_no_import` | type, control_number |
| `materialflarebindings` | flare_number, material_name |
| `managedmaterials` | name, type (flexmesh_standard / flexmesh_transparent / mesh_standard / mesh_transparent), options {double_sided}, diffuse, damaged_diffuse, specular maps |
| `submeshes` | backmesh flag, texcoords[(node, u, v)], cab triangles[(nodes[3], options)] |
| `submesh_groundmodel` | ground model name |
| `exhausts` | reference node, direction node, particle system name |
| `particles` | emitter node, reference node, particle system name |
| `soundsources` | node, sound script name |
| `soundsources2` | node, mode (−2 always, −1 exterior only, ≥0 cinecam index), script name |
| `videocameras` | ref/left/bottom nodes, alt ref & orientation nodes (optional), offset, rotation, fov, texture w/h, clip near/far, role, mode, material, camera name |
| `extcamera` | mode (classic/cinecam/node), node |
| `wings` | nodes[8], texcoords[8], control surface ('n'), chord_point, min/max deflection, airfoil, efficacy_coef (1.0) |
| `airbrakes` | ref/x/y/additional nodes, offset, width, height, max inclination angle, texcoords x1/x2/y1/y2, lift coefficient (1.0) |
| `turboprops2` (also `turboprops`) | reference, axis, blade tips[4] (last two optional), couple node, power (kW), airfoil |
| `pistonprops` | same as turboprops + pitch |
| `turbojets` | front, back, side nodes, reversible (0/1), dry thrust, wet thrust, front/back diameter, nozzle length |
| `fusedrag` | front & rear node, autocalc (false), approximate width, airfoil ("NACA0009.afl"), area coefficient (1.0) |
| `screwprops` | prop, back, top nodes, power |
| `rotators` / `rotators2` | axis nodes[2], base plate nodes[4], rotating plate nodes[4], rate, spin-left key, spin-right key, inertia, engine_coupling (1.0), needs_engine (false); rotators2 adds rotating_force (10,000,000), tolerance (0), description |
| `guisettings` | key, value |
| `customdashboardinputs` | name, data type (bool/int/float/string) |
| `help` | material name |
| `author`, `fileinfo`, `guid`, `fileformatversion`, `description`, `default_skin`, `assetpacks`, `scripts` | metadata; see parser |

### Option-letter enums (file values)

- **Engine type** `c` car, `e` electric car, `t` truck.
- **Differential** `o` open, `l` locked, `s` split, `v` viscous.
- **Wing control surface** `n` none, `a`/`b` right/left aileron, `f` flap, `e` elevator, `r` rudder, `S`/`T` right/left stabilator, `c`/`d` right/left elevon, `g`/`h` right/left flaperon, `U`/`V` right/left taileron, `i`/`j` right/left ruddervator.
- **Cab options** `n` none, `c` contact, `b` buoyant, `p` 10× tougher, `u` invulnerable, `s` buoyant without drag, `r` buoyant drag only, and composites `D` = c+b, `F` = p+b, `S` = u+b.
- **Trigger options** `i` invisible, `c` command-style bounds, `x` start disabled, `b` key blocker, `B` trigger blocker, `A` inverted trigger blocker, `s` command-number switch, `h` unlocks hook group, `H` locks hook group, `t` continuous (0..1 output), `E` engine trigger.
- **Special props** (recognised from the mesh file name): mirror left/right, dashboard (left-hand / `-rh` right-hand), aero propeller spinner / blade, driver seat / second seat, beacon, red beacon, lightbar.
- **Animation sources/modes** — see the parser's `add_animation` grammar.

`ManagedMaterial.TypeToStr` gives the file spelling of the material type.
