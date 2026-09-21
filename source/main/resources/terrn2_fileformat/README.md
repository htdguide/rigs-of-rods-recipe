# source/main/resources/terrn2_fileformat

> The `.terrn2` format: terrain descriptor.

The `.terrn2` terrain descriptor: name, geometry config, environment, and lists of object, script and preset files. A pure parser/serializer — it produces a document and knows nothing about the game objects built from it. The format is owned by existing community content, so a rebuild must accept existing files unchanged (see [Data and persistence](../../../../SYSTEM-REQUIREMENTS.md#5-data-and-persistence)).

## Reading order

1. [`Terrn2FileFormat.h`](Terrn2FileFormat.h.md) — the document records
2. [`Terrn2FileFormat.cpp`](Terrn2FileFormat.cpp.md) — parsing and writing rules
