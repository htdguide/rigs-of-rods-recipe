# source/main/resources/otc_fileformat

> The `.otc` format: terrain geometry config.

The `.otc` heightmap-terrain configuration: world size, paging grid, rendering options and per-page texture layers. A pure parser/serializer — it produces a document and knows nothing about the game objects built from it. The format is owned by existing community content, so a rebuild must accept existing files unchanged (see [Data and persistence](../../../../SYSTEM-REQUIREMENTS.md#5-data-and-persistence)).

## Reading order

1. [`OTCFileFormat.h`](OTCFileFormat.h.md) — the document records
2. [`OTCFileFormat.cpp`](OTCFileFormat.cpp.md) — parsing and writing rules
