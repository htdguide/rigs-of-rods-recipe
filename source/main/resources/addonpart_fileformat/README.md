# source/main/resources/addonpart_fileformat

> The `.addonpart` format: add-on parts.

Turns an `.addonpart` file into an extra vehicle module plus tuning tweaks, and detects conflicts between parts. A pure parser/serializer — it produces a document and knows nothing about the game objects built from it. The format is owned by existing community content, so a rebuild must accept existing files unchanged (see [Data and persistence](../../../../SYSTEM-REQUIREMENTS.md#5-data-and-persistence)).

## Reading order

1. [`AddonPartFileFormat.h`](AddonPartFileFormat.h.md) — the document records
2. [`AddonPartFileFormat.cpp`](AddonPartFileFormat.cpp.md) — parsing and writing rules
