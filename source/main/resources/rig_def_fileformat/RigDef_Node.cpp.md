# source/main/resources/rig_def_fileformat/RigDef_Node.cpp

> Constructors and debug formatting for node ids and references.

**Needs** — [`RigDef_Node.h`](RigDef_Node.h.md) · [`Application.h`](../../Application.h.md)
**Used by** — callers of [`RigDef_Node.h`](RigDef_Node.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Trivial helpers behind [`RigDef_Node.h`](RigDef_Node.h.md).

## State

See header twin.

## `NodeId.SetNum` / `NodeId.setStr` / `Invalidate`

**Contract** — setting a number stores it and its decimal text and marks the id NUMBERED+VALID; setting text stores it with number 0 and marks NAMED+VALID; invalidating clears everything.

## `NodeRef` construction / `Invalidate`

**Contract** — a reference is built from its text, parsed number, the flag set chosen by the parser, and the source line; invalidating clears it.

## `ToString`

**Contract** — diagnostic text: `Node::Ref(id:<text>, src line:<n or ?>, import flags:[…], regular flags:[…])` and `Node::Id(<text> NUMBERED|NAMED)`.
