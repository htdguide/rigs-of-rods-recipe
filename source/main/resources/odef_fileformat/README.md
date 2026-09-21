# source/main/resources/odef_fileformat

> The `.odef` format: object definitions.

The `.odef` terrain-object definition: mesh, scale, collision boxes and meshes, lights, particles, sounds, animations. A pure parser/serializer — it produces a document and knows nothing about the game objects built from it. The format is owned by existing community content, so a rebuild must accept existing files unchanged (see [Data and persistence](../../../../SYSTEM-REQUIREMENTS.md#5-data-and-persistence)).

## Reading order

1. [`ODefFileFormat.h`](ODefFileFormat.h.md) — the document records
2. [`ODefFileFormat.cpp`](ODefFileFormat.cpp.md) — parsing and writing rules
