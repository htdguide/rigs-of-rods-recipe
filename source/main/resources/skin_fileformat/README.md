# source/main/resources/skin_fileformat

> The `.skin` format: vehicle skins.

The `.skin` document: texture and material substitutions that restyle a vehicle. A pure parser/serializer — it produces a document and knows nothing about the game objects built from it. The format is owned by existing community content, so a rebuild must accept existing files unchanged (see [Data and persistence](../../../../SYSTEM-REQUIREMENTS.md#5-data-and-persistence)).

## Reading order

1. [`SkinFileFormat.h`](SkinFileFormat.h.md) — the document records
2. [`SkinFileFormat.cpp`](SkinFileFormat.cpp.md) — parsing and writing rules
