# source/main/resources/rig_def_fileformat/RigDef_Prerequisites.h

> Forward declarations for the truck-format document types.

**Needs** — nothing
**Used by** — [`gfx/GfxActor.h`](../../gfx/GfxActor.h.md) · [`physics/Actor.h`](../../physics/Actor.h.md) · [`physics/ActorManager.h`](../../physics/ActorManager.h.md) · [`physics/flex/FlexBody.h`](../../physics/flex/FlexBody.h.md) · [`physics/flex/FlexFactory.h`](../../physics/flex/FlexFactory.h.md) · [`RigDef_Node.h`](RigDef_Node.h.md) · [`RigDef_Parser.h`](RigDef_Parser.h.md) · [`RigDef_SequentialImporter.h`](RigDef_SequentialImporter.h.md)
**Tier floor** — T4

## Purpose

Lets other headers mention truck-format record types (`Beam`, `Engine`, `Flexbody`, …) and the `Parser` / `Validator` / `SequentialImporter` classes without including [`RigDef_File.h`](RigDef_File.h.md). No behaviour; a language without header files needs nothing here.

## State

Stateless.
