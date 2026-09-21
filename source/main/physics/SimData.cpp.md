# source/main/physics/SimData.cpp

> Constructors of the records in `SimData.h` and the `ActorSimAttr` name table.

**Needs** — [`SimData.h`](SimData.h.md) · [`Actor.h`](Actor.h.md) · [`resources/CacheSystem.h`](../resources/CacheSystem.h.md) · [`resources/tuneup_fileformat/TuneupFileFormat.h`](../resources/tuneup_fileformat/TuneupFileFormat.h.md)
**Used by** — callers of [`SimData.h`](SimData.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Exists only because the source language needs out-of-line constructors for records holding shared handles. The one piece of content is that ties start untied (`no_self_lock`, `tied`, `tying` all false) and `ActorSimAttrToString`, which returns each attribute's identifier text (`"TC_RATIO"`, `"ENGINE_SHIFTDOWN_RPM"`, …).

## State

Stateless.
