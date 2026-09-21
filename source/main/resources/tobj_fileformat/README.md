# source/main/resources/tobj_fileformat

> The `.tobj` format: terrain object placements.

The `.tobj` terrain-objects list: placed objects, pre-spawned vehicles, roads, trees and grass. A pure parser/serializer — it produces a document and knows nothing about the game objects built from it. The format is owned by existing community content, so a rebuild must accept existing files unchanged (see [Data and persistence](../../../../SYSTEM-REQUIREMENTS.md#5-data-and-persistence)).

## Reading order

1. [`TObjFileFormat.h`](TObjFileFormat.h.md) — the document records
2. [`TObjFileFormat.cpp`](TObjFileFormat.cpp.md) — parsing and writing rules
