# source/main/resources/rig_def_fileformat/RigDef_File.cpp

> Default values of the section records and the empty document.

**Needs** — [`RigDef_File.h`](RigDef_File.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`resources/CacheSystem.h`](../CacheSystem.h.md) · [`physics/SimConstants.h`](../../physics/SimConstants.h.md)
**Used by** — callers of [`RigDef_File.h`](RigDef_File.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Holds the defaults that are not inline in the header (all are listed in the header twin's table), the root module name `_Root_`, and two tiny helpers.

## State

```text
CONST ROOT_MODULE_NAME = "_Root_"
```

## `Document()`

**Contract** — all flags false; creates the root module.

## `Animation.AddMotorSource(source, motor)`

**Contract** — appends `{source flag, engine index}` to the animation's motor-source list.

## `ManagedMaterial.TypeToStr(type)`

**Contract** — `flexmesh_standard`, `flexmesh_transparent`, `mesh_standard`, `mesh_transparent`; empty for invalid.
