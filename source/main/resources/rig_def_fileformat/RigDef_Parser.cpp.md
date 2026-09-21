# source/main/resources/rig_def_fileformat/RigDef_Parser.cpp

> Line-by-line reader of the truck format: tokenisation, keyword recognition, block state, directives, and the exact column syntax of every section.

**Needs** — [`RigDef_Parser.h`](RigDef_Parser.h.md) · [`RigDef_File.h`](RigDef_File.h.md) · [`RigDef_Regexes.h`](RigDef_Regexes.h.md) · [`RigDef_SequentialImporter.h`](RigDef_SequentialImporter.h.md) · [`Application.h`](../../Application.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`gui/DashBoardManager.h`](../../gui/DashBoardManager.h.md) (dashboard data-type ids) · [`physics/SimConstants.h`](../../physics/SimConstants.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md)
**Used by** — callers of [`RigDef_Parser.h`](RigDef_Parser.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

This file *is* the truck format specification in executable form. Tens of thousands of community vehicles depend on its tolerances, including its bugs, so a rebuild must reproduce the grammar below — especially how it recovers from malformed lines (skip the line with a warning, never abort the file). All messages go to the console as area ACTOR, formatted `"<file>:<line> (<keyword>): <message>"`.

## State

See [`RigDef_Parser.h`](RigDef_Parser.h.md).

## Reading lines

```text
FUNCTION process_raw_line(raw)          # raw is at most 1999 bytes; longer lines are truncated
  strip leading spaces/tabs
  IF empty: line_number += 1; RETURN
  IF first char is ';' or '/'           # comment: ';' anywhere at line start, or any '/' (so '//' and '/x')
    pending_comment += raw + "\n"       # attached to the next element parsed (see FlushPendingDocComment)
    line_number += 1; RETURN
  line = raw with invalid UTF-8 replaced by '?'
  process_current_line(line)
  line_number += 1
```

**Notes** — trailing whitespace and inline comments are **not** stripped. A line such as `1, 2, 3 ;note` tokenises `;note` as an argument; numeric parses of such garbage yield 0 (which is why some optional arguments explicitly ignore 0, e.g. cinecam node mass). Comment lines inside `description` / `comment` blocks are still treated as comments.

## Tokenising

**Contract** — separators are space, tab, `,`, `:` and `|`; any run of separators counts as one; at most 100 tokens per line. Blocks `comment` and `description` are not tokenised. Several sections re-split the raw line themselves (noted in the table).

## `process_current_line`

```text
FUNCTION process_current_line(line)
  IF document.name is empty: document.name = line; RETURN      # first data line = vehicle title
  IF block not in {COMMENT, DESCRIPTION}: tokenise(line)
  keyword = INVALID
  IF line starts with an ASCII letter: keyword = identify_keyword(line)
  CASE keyword OF
    flag keywords (disabledefaultsounds, enable_advanced_deformation, forwardcommands,
      hideInChooser, importcommands, lockgroup_default_nolock, rescuer, rollon,
      slidenode_connect_instantly):             set the document flag; RETURN
    end_section:                                switch to root module; RETURN
    inline directives (add_animation, AntiLockBrakes, author, backmesh, cruisecontrol,
      default_skin, detacher_group, extcamera, fileformatversion, fileinfo,
      flexbody_camera_mode, forset, forvert, guid, prop_camera_mode, section,
      set_beam_defaults, set_beam_defaults_scale, set_collision_range, set_default_minimass,
      set_inertia_defaults, set_managedmaterials_options, set_node_defaults,
      set_skeleton_settings, speedlimiter, submesh, submesh_groundmodel, TractionControl):
                                                parse this line now; block unchanged; RETURN
    end, end_comment, end_description:          begin_block(NONE); RETURN
    envmap, hookgroup, nodecollision, rigidifiers:   obsolete, ignore line; RETURN
    any other keyword:                          begin_block(keyword); RETURN
    INVALID:                                    fall through (data line)
  dispatch the line to the parser of the current block (table below); lines with no current
  block, or in a block without a parser (sectionconfig, set_shadows, SlopeBrake…), are ignored
```

**Notes** — because keyword recognition runs on every line (description included), a description line that happens to equal a keyword ends the description. `sectionconfig`, `set_shadows` and `SlopeBrake` are recognised keywords with no parser: they open a block whose lines are silently dropped until the next keyword.

## `IdentifyKeyword(line)`

**Contract** — case-insensitive. A **block keyword** matches only if it is alone on the line (trailing blanks allowed); an **inline keyword** matches only if followed by at least one separator and then anything; `forset` also matches with no separator (`forset1-5`). Exact whole-word matching means `nodes` ≠ `nodes2`, `set_beam_defaults` ≠ `set_beam_defaults_scale`. The recogniser returns the keyword's position in a fixed alphabetical list, which is also its numeric id (see [`RigDef_Regexes.h`](RigDef_Regexes.h.md)).

## `begin_block(keyword)`

```text
FUNCTION begin_block(kw)
  IF kw == NONE                        # also the effect of 'end' and of starting 'submesh'
    IF a submesh is being built: append it to the module; clear
    IF a camera rail is being built
      IF it has nodes: append it and clear   ELSE warn "Empty section 'camerarail', ignoring..."
  ELSE IF kw == CAMERARAIL
    begin_block(NONE); start a new camera rail     # each 'camerarail' keyword starts a separate rail
  current_block = kw
```

## Modules

**Contract** — `section <unused> <name> [...]` switches the current module to `name` (creating it on first use; re-entering the current module is an error and ignored); only the first name is used. `end_section` returns to the root module (an error if already there). Switching modules flushes pending submesh/camera rail. Module membership is by name across the file, so the same name used twice appends to the same module.

## Defaults directives

All defaults are "sticky" snapshots: each new directive creates a fresh defaults object; lines parsed afterwards reference it; lines parsed before keep the old one.

- `set_beam_defaults spring, [damp], [deform], [break], [diameter], [material], [plastic_coef]` — copies the current defaults and overrides given values; any value < 0 becomes the game default (spring 9,000,000; damp 12,000; deform 400,000; break 1,000,000; diameter 0.05; plastic 0). Records whether `enable_advanced_deformation` had already been seen, and marks the defaults user-defined.
- `set_beam_defaults_scale spring, [damp], [deform], [break]` — requires 4 values after the keyword; copies the current defaults with new scale factors.
- `set_node_defaults load_weight, [friction], [volume], [surface], [options]` — each negative value falls back to the **game** defaults (not the previous user values): load weight −1, friction 1, volume 1, surface 1.
- `set_inertia_defaults start_delay, [stop_delay], [start_fn], [stop_fn]` — any negative delay **resets** to game inertia defaults.
- `set_default_minimass kg` — minimum mass for nodes defined afterwards.
- `set_managedmaterials_options d` — double-sided = first character of the argument ≠ `'0'`.
- `detacher_group n` / `detacher_group end` — sets the group given to subsequent beams, shocks, hydros, commands, triggers, ropes, ties, animators, nodes; `end` resets to 0.
- `set_collision_range r`, `set_skeleton_settings range, [thickness]` (negative → 150 m / 0.01 m; there is only ever one skeleton-settings record per module, later lines overwrite it).

Parser state is reset per file: beam defaults = game defaults, node defaults = game defaults, inertia = game defaults, detacher group 0, managed-material options default.

## Section syntax

Columns are separated by any separator. `N` = node reference, `?N` = nullable node (`-1` means none), `R` = rigidity node (`9999` means none), `f` float, `i` integer, `s` string, `[x]` optional. "min" is the minimum number of columns; shorter lines are skipped with "Not enough arguments (got X, Y needed), skipping line".

| Section | Columns | min | Notes |
|---|---|---|---|
| `nodes` | i id, f x, f y, f z, [s options], [f load_weight] | 4 | id parsed as unsigned; load weight only honoured with option `l` (else warning) |
| `nodes2` | s name, f x, f y, f z, [options], [load] | 4 | named node |
| `beams` | N, N, [s options i/r/s/v], [i support_break_factor] | 2 | break factor only with `s`; values ≤ 0 mean 0 (infinite) |
| `shocks` | N, N, f spring, f damp, f short, f long, f precomp, [opts i/L/R/m/n/v] | 7 | |
| `shocks2` | N, N, spring_in, damp_in, prog_spring_in, prog_damp_in, spring_out, damp_out, prog_spring_out, prog_damp_out, short, long, precomp, [opts i/s/m/M/n/v] | 13 | |
| `shocks3` | N, N, spring_in, damp_in, damp_in_slow, split_in, damp_in_fast, spring_out, damp_out, damp_out_slow, split_out, damp_out_fast, short, long, precomp, [opts i/m/M/n/v] | 15 | |
| `hydros` | N, N, f factor, [opts], [inertia ×4] | 3 | no options → `n` (steering input). Options: `n` steering, `j` invisible, `i` = invisible + (if first letter) steering, `s` disable at high speed, `a` aileron, `r` rudder, `e` elevator, `u` aileron+elevator, `v` −aileron+elevator, `x` aileron+rudder, `y` −aileron+rudder, `g` elevator+rudder, `h` −elevator+rudder |
| `commands` | N, N, f rate, f max_contract, f max_extend, i key_contract, i key_extend, [opts], [desc], [inertia ×4], [f affect_engine], [bool needs_engine], [bool plays_sound] | 7 | lengthen rate = shorten rate |
| `commands2` | N, N, f shorten_rate, f lengthen_rate, then as `commands` | 8 | |
| `triggers` | N, N, f contract_limit, f expand_limit, i short_action, i long_action, [opts], [f boundary_timer] | 6 | |
| `animators` | raw split on `,`: N, N, f factor, `opt|opt|…` | 4 | options below |
| `ties` | N, f max_reach, f auto_shorten_rate, f min_len, f max_len, [opts n/v/i/s], [f max_stress], [i group] | 5 | |
| `ropes` | N root, N end, [s: `i` = invisible] | 2 | |
| `ropables` | N, [i group], [i multilock: 1 = yes] | 1 | |
| `fixes`, `contacters` | N | 1 | `fixes` reads the first token without checking count |
| `hooks` | N, then any of: `hookrange f`, `speedcoef f`, `maxforce f`, `timer f`, `hookgroup`/`hgroup i`, `lockgroup`/`lgroup i`, `shortlimit`/`short_limit f`, flags `selflock`/`self-lock`/`self_lock`, `autolock`(…), `nodisable`(…), `norope`(…), `visible`/`vis` | 1 | valued options need a following token; unknown words warn |
| `lockgroups` | i number, N… | 2 | |
| `slidenodes` | raw split on `,` and space: N slide, then rail nodes until the first option; options: `S<f>` spring, `B<f>` break force, `T<f>` tolerance, `R<f>` attach rate, `G<n>` rail group, `D<f>` max attach distance, `Ca` / `Cf` / `Cs` / `Cn` constraint | 2 | option letters case-insensitive |
| `railgroups` | raw split on `,`: i id, N… | 3 | |
| `cameras` | N center, N back, N left | 3 | |
| `cinecam` | f x, f y, f z, N ×8, [f spring], [f damp], [f node_mass] | 11 | node mass ≤ 0 ignored |
| `camerarail` | N | — | appended to the current rail |
| `collisionboxes` | raw split on `,`: N… | — | |
| `wheels` | f radius, f width, i rays, N, N, R, i braking, i propulsion, N arm, f mass, f spring, f damp, s face_mat, s band_mat | 14 | |
| `wheels2` | f rim_r, f tyre_r, f width, i rays, N, N, R, i brake, i prop, N arm, f mass, f rim_spring, f rim_damp, f tyre_spring, f tyre_damp, s face_mat, s band_mat | 17 | |
| `meshwheels`, `meshwheels2` | f tyre_r, f rim_r, f width, i rays, N, N, R, i brake, i prop, N arm, f mass, f spring, f damp, c side, s mesh, s material | 16 | |
| `flexbodywheels` | f tyre_r, f rim_r, f width, i rays, N, N, R, i brake, i prop, N arm, f mass, f tyre_spring, f tyre_damp, f rim_spring, f rim_damp, c side, [s rim_mesh], [s tyre_mesh] | 16 | |
| `wheeldetachers` | i wheel, i group | 2 | |
| `engine` | f shift_down_rpm, f shift_up_rpm, f torque, f global_ratio, f reverse_ratio, f neutral_ratio, f gear… | 6 | gears end at the first negative value; zero gears → error, line dropped |
| `engoption` | f inertia, [c type], [clutch_force], [shift_time], [clutch_time], [post_shift_time], [stall_rpm], [idle_rpm], [max_idle_mix], [min_idle_mix], [braking_torque] | 1 | note: file order is stall *then* idle rpm |
| `engturbo` | i version, f inertia_factor, i n_turbos, f p1, [p2…p11] | 4 | n_turbos > 4 → 4 with warning |
| `torquecurve` | raw split on `,`: either one token (predefined curve name) or `f rpm_fraction, f torque_percent` | — | 3+ tokens → error; one curve record per module |
| `brakes` | f force, [f parking_force] | 1 | |
| `AntiLockBrakes` (inline) | text after the 15-char keyword split on `,`: f regulation_force, i min_speed, [f pulse_per_sec], [`mode: on&off&nodash&notoggle`] | 2 | pulse only read if there are ≥4 tokens; tokens after that must be `mode:` or reset attrs to defaults with error "missing mode" |
| `TractionControl` (inline) | as above: f regulation_force, f wheel_slip, [f fade_speed], [f pulse], [`mode:`…] | 2 | mode tokens start at index 4 |
| `cruisecontrol` (inline) | f min_speed, i autobrake | 3 incl. keyword | |
| `speedlimiter` (inline) | f max_speed | 2 incl. keyword | sets enabled |
| `axles` | raw split on `,`; each part is `w1(N N)`, `w2(N N)` or `d(olsv…)` | — | any unparsable part drops the whole line |
| `interaxles` | raw split on `,`: i axle_a (1-based), i axle_b, `d(…)` | 2 | stored 0-based |
| `transfercase` | i axle_a (1-based), i axle_b (1-based), [i has_2wd], [i has_2wd_lo], [f extra gear ratios…] | 2 | stored 0-based; gear list starts with 1.0 |
| `globals` | f dry_mass, f cargo_mass, [s material] | 2 | |
| `minimass` | f kg, [c option n/l] | 1 | ends the block after one line |
| `props` | N ref, N x, N y, f ox, f oy, f oz, f rx, f ry, f rz, s mesh, [special args] | 10 | beacon props with ≥14 columns: s flare_material, f r, f g, f b; dashboard props: [s wheel_mesh], [f ox f oy f oz], [f rotation_angle] |
| `add_animation` (inline) | text after 14 chars split on `,`: f ratio, f lower_limit, f upper_limit, then items | 4 | applies to the last prop; see below |
| `prop_camera_mode` (inline) | i mode | 2 incl. keyword | applies to last prop |
| `flexbodies` | N ref, N x, N y, f ox, f oy, f oz, f rx, f ry, f rz, s mesh | 10 | must be followed by `forset` |
| `forset` (inline) | see below | — | applies to the last flexbody |
| `forvert` (inline) | N ref, N x, N y, `verts: a-b, c, …` | 4 incl. keyword | one binding per listed vertex; ranges inclusive |
| `flexbody_camera_mode` (inline) | i mode | 2 incl. keyword | applies to last flexbody |
| `flares` | N ref, N x, N y, f ox, f oy, [c type], [i control / s dash_link], [i blink_ms], [f size], [s material] | 5 | the 6th column meaning depends on type: `u` → control number, `d` → dashboard link name |
| `flares2` | as `flares` with f oz after oy | 6 | |
| `flares3` | as `flares2`, positions fixed | 5 | also captures current inertia defaults |
| `flaregroups_no_import` | c type, [i control 1–10] | 1 | out-of-range control only warns |
| `materialflarebindings` | i flare_number, s material | 2 | |
| `managedmaterials` | s name, s type, s diffuse, [s spec or damaged_diffuse], [s spec] | 2 (3 if type valid) | mesh types: 4th = specular; flexmesh types: 4th = damaged diffuse, 5th = specular; a texture name starting with `-` means none; invalid type → line dropped |
| `submesh` (inline) | — | — | flushes and starts a new submesh |
| `backmesh` (inline) | — | — | marks current submesh; error if none |
| `texcoords` | N, f u, f v | 3 | needs a current submesh |
| `cab` | N, N, N, [opts] | 3 | needs a current submesh |
| `submesh_groundmodel` (inline) | s name | 2 incl. keyword | |
| `exhausts` | N ref, N direction, (ignored), [s particle] | 2 | |
| `particles` | N emitter, N reference, s system | 3 | |
| `soundsources` | N, s script | 2 | |
| `soundsources2` | N, i mode, s script | 3 | mode < −2 → −2 with error |
| `videocamera` | N ref, N left, N bottom, ?N alt_ref, ?N alt_orient, f ox, f oy, f oz, f rx, f ry, f rz, f fov, i tex_w, i tex_h, f clip_min, f clip_max, i role, i mode, s material, [s name] | 19 | role must be −1, 0, 1, 2 else "videocamera will not work" |
| `extcamera` (inline) | s `classic`/`cinecam`/`node`, [N] | 2 incl. keyword | unknown → classic |
| `wings` | N ×8, f ×8 texcoords, [c surface], [f chord], [f min_defl], [f max_defl], [s airfoil], [f efficacy] | 16 | |
| `airbrakes` | N ref, N x, N y, N additional, f ox, f oy, f oz, f width, f height, f max_angle, f tx1, f ty1, f tx2, f ty2 | 14 | note texcoord order tx1, ty1, tx2, ty2 |
| `turboprops` | N ref, N axis, N tip1, N tip2, ?N tip3, ?N tip4, f power_kW, s airfoil | 8 | |
| `turboprops2` | as above with ?N couple after tip4 | 9 | |
| `pistonprops` | N ref, N axis, N tip1, N tip2, ?N tip3, ?N tip4, ?N couple, f power_kW, f pitch, s airfoil | 10 | |
| `turbojets` | N front, N back, N side, i reversible, f dry_thrust, f wet_thrust, f front_d, f back_d, f nozzle_len | 9 | |
| `fusedrag` | N front, N rear, then either `autocalc` [f area_coef] [s airfoil] or f width [s airfoil] | 3 | |
| `screwprops` | N prop, N back, N top, f power | 4 | |
| `rotators` | N axis ×2, N base ×4, N rotating ×4, f rate, i key_left, i key_right, [inertia ×4], [f engine_coupling], [bool needs_engine] | 13 | |
| `rotators2` | as `rotators` but after the keys: f force, f tolerance, s description, then inertia… | 16 | |
| `guisettings` | s key, s value | 2 | |
| `customdashboardinputs` | s name, s type `bool`/`int`/`float`/`string` | 2 | invalid type → error, dropped |
| `help` | whole line = material name | — | |
| `description` | whole line appended | — | until `end_description` |
| `author` (inline) | s type, [i forum_id], [s name], [s email] | 2 incl. keyword | presence of forum id recorded |
| `fileinfo` (inline) | s unique_id, [i category], [i file_version] | 2 incl. keyword | |
| `guid` (inline) | s guid | 2 incl. keyword | |
| `fileformatversion` (inline) | i version | 2 incl. keyword | |
| `default_skin` (inline) | s name | 2 incl. keyword | underscores replaced by spaces |
| `assetpacks` | s filename | 1 | |
| `scripts` | s filename | 1 | |

### Scalar parsing

- floats: locale-independent; unparsable → 0.
- ints: base-10 prefix parse; no digits → 0 with error "is not valid integer"; trailing garbage → warning, prefix value used.
- bools: `true/yes/1` / `false/no/0`.
- braking: 0–4 else 0 with error; propulsion: 0–2 else 0 with error.
- wheel side: `l` / `r`; anything else → `l` with warning.
- flare type: one of `f h g t b R s l r u d`; else `f` with warning.
- wing surface: listed letters; else `n`.
- engine type: `t c e`; else `t`.

### `add_animation` items

After ratio, lower and upper limit, each comma-separated item is either a single word or `key: value`:

- words: `autoanimate`, `noflip`, `bounce`, `eventlock`;
- `mode: m|m…` with `x-rotation`, `y-rotation`, `z-rotation`, `x-offset`, `y-offset`, `z-offset`;
- `event: NAME` (stored upper-cased; the input event that drives the animation);
- `link: NAME` (dashboard data link);
- `source: s|s…` with `airspeed`, `vvi`, `altimeter100k`, `altimeter10k`, `altimeter1k`, `aoa`, `flap`, `airbrake`, `roll`, `pitch`, `brakes`, `accel`, `clutch`, `speedo`, `tacho`, `turbo`, `parking`, `shifterman1`, `shifterman2`, `sequential`, `shifterlin`, `autoshifterlin`, `torque`, `heading`, `difflock`, `rudderboat`, `throttleboat`, `steeringwheel`, `aileron`, `elevator`, `rudderair`, `permanent`, `event`, `dashboard`, `signalstalk`, `gearreverse`, `gearneutral`; or an engine-indexed source `throttle<N>`, `rpm<N>`, `aerotorq<N>`, `aeropit<N>`, `aerostatus<N>`, `gear<N>`.

Invalid items warn and are skipped. **Quirk:** the engine-indexed sources are detected from the whole `source:` value, not from the individual `|`-separated item, so they only work when they are the only source listed.

### `animators` options

`|`-separated: `vis`, `inv`, `airspeed`, `vvi`, `altimeter100k`, `altimeter10k`, `altimeter1k`, `aoa`, `flap`, `airbrake`, `roll`, `pitch`, `brakes`, `accel`, `clutch`, `speedo`, `tacho`, `turbo`, `parking`, `shifterman1`, `shifterman2`, `sequential`, `shifterlin`, `torque`, `difflock`, `rudderboat`, `throttleboat`; `shortlimit: f`, `longlimit: f`; engine-indexed `throttle<d>`, `rpm<d>`, `aerotorq<d>`, `aeropit<d>`, `aerostatus<d>` (single digit, 1-based, stored 0-based).

### `commands` option letters

`n` filler, `i` invisible, `r` rope (no compression), `f` not faster, `c` auto-centre, `p` one-press, `o` one-press with centring. Only one of `c`, `p`, `o` may be active: the **first** of them appearing in the string wins; the others are cleared with a warning.

## `ProcessForsetLine(flexbody, line)`

**Contract** — reproduces the legacy parser's quirks exactly; flexbody vertex binding in thousands of mods depends on them.

```text
FUNCTION process_forset(flexbody, line)
  pos = 6                                           # skip "forset"
  skip any ' ', ':' or ',' at pos
  LOOP
    item_end = next '-' or ',' or end of line
    a = leading unsigned integer of line[pos..item_end] (0 if none)
    IF the char at item_end is '-'
      pos = item_end + 1
      item_end = next ',' or end
      b = leading unsigned integer of line[pos..item_end] (0 if none)
      ADD RANGE (a, b)
    ELSE
      ADD SINGLE a
    IF item_end was end of line: STOP
    pos = item_end + 1
```

Consequences: whitespace-separated numbers keep only the first (`1 2, 3` → 1, 3); a trailing separator adds node 0; `a-b-c` keeps `a-b`; a leading `-` gives range `0-n`; named nodes in `forset` do not work (they parse as 0). References produced are numbered-import-valid only.

## `FlushPendingDocComment(count, keyword)`

**Contract** — after an element is appended, attaches any accumulated comment lines to it (keyword + index) and clears the pending text. Directives do not flush, so a comment before a directive attaches to the next element.

## `Prepare` / `Finalize` / `ProcessOgreStream`

**Contract** — prepare resets all state and enables the sequential importer; `ProcessOgreStream(stream, group)` reads lines of up to 1999 bytes (a read error logs "Could not read truck file" and stops); finalize flushes the current block and runs the [sequential importer](RigDef_SequentialImporter.cpp.md) over the document.

## Node references while parsing

**Contract** — while the importer is enabled (always, at present), every reference is created with both states: import-valid with number = |integer value of the text| (plus "check named first" if any `nodes2` line appeared earlier in the file), and regular-valid-named with the text. Numbered nodes are registered with the importer in order; named nodes by name; wheels and cinecams register their generated nodes (see importer twin).

## `IdentifySpecialProp(mesh_name)`

**Contract** — substring `leftmirror` → mirror left; `rightmirror` → mirror right; `dashboard-rh` → right-hand dashboard; `dashboard` → left-hand dashboard; then case-insensitive prefixes `spinprop` → propeller spinner, `pale` → propeller blade, `seat` → driver seat (note `seat2` is shadowed by `seat` and never matches), `beacon`, `redbeacon`, `lightb` → lightbar. Checked in this order.
