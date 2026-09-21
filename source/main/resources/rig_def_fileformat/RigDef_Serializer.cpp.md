# source/main/resources/rig_def_fileformat/RigDef_Serializer.cpp

> Canonical output order and formatting for truck files.

**Needs** — [`RigDef_Serializer.h`](RigDef_Serializer.h.md) · [`RigDef_File.h`](RigDef_File.h.md) · [`physics/SimConstants.h`](../../physics/SimConstants.h.md)
**Used by** — callers of [`RigDef_Serializer.h`](RigDef_Serializer.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Produces a file the [parser](RigDef_Parser.cpp.md) can read back, in a fixed section order rather than the original order. Each section writes its keyword, then one line per record using the column orders of the parser's syntax table (comma-space separated, numbers right-aligned to the configured widths), preceded by any attached doc comment, and a blank line after.

## State

See header twin.

## `Serialize`

```text
FUNCTION serialize()
  write banner: 4 comment lines naming the project and the format documentation URL
  write document.name, blank line
  write description, authors, fileinfo, guid                  # from the root module
  write file flags (enable_advanced_deformation, hideInChooser, slidenode_connect_instantly,
                    lockgroup_default_nolock, rollon, rescuer, disabledefaultsounds,
                    forwardcommands, importcommands), each followed by a blank line
  serialize_module(root)
  FOR EACH user module: serialize_module(module)
  write "end"

FUNCTION serialize_module(m)
  IF m is not root: write "section -1 " + m.name
  managedmaterials (+ options), globals,
  nodes, beams, cameras, shocks, shocks2, hydros, commands2, slidenodes, ties, ropes, fixes,
  meshwheels, meshwheels2, wheels, wheels2, flexbodywheels,
  engine, engoption, brakes, AntiLockBrakes, TractionControl, torquecurve, cruisecontrol, speedlimiter,
  axles, transfercase, interaxles,
  cinecam, animators, contacters, triggers, lockgroups, hooks, railgroups, ropables, particles,
  collisionboxes, flares2, materialflarebindings, props (+ add_animation), submesh, submesh_groundmodel,
  exhausts, guisettings, set_skeleton_settings, videocamera, extcamera, soundsources, soundsources2,
  customdashboardinputs, wings, airbrakes, turboprops, fusedrag, pistonprops, turbojets, screwprops
  IF m is not root: write "endsection"
```

## Presets

**Contract** — before writing a line that carries node/beam defaults or default minimass, if the object differs from the last one written, emit the corresponding directive first. Beam defaults are written as `set_beam_defaults spring, damp, deform, break, diameter, material, plastic` with values equal to the game default written as `-1`; a missing defaults object is written as all `-1` with a blank material.

## Nodes

**Contract** — `nodes` then every node as `id, x, y, z, options, [load_weight]`; then, if any node is named, `nodes2` and the named nodes again.

## Known defects

A rebuild should not copy these; they are listed so a comparison against the original's output is not surprising:

- modules are closed with `endsection`, which the parser does not recognise (it expects `end_section`);
- named nodes are written in both the `nodes` and the `nodes2` block;
- the module writer's call list omits several sections even though writers exist for some of them (`shocks3`, `rotators`, `rotators2`, `flexbodies`, `help` have writer functions; `flares3`, `forvert`, `engturbo`, `wheeldetachers`, `assetpacks`, `scripts`, `default_skin`, `set_collision_range`, detacher groups have none) — a complete serializer must cover every row of the parser's syntax table.

The practical contract is therefore narrower than a full round-trip: output is readable for single-module vehicles using the common sections.
