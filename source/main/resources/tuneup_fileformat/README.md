# source/main/resources/tuneup_fileformat

> The `.tuneup` format: saved vehicle tunings.

The vehicle tuning record: which add-on parts are fitted, what they change, and what the player forced or protected. A pure parser/serializer — it produces a document and knows nothing about the game objects built from it. The format is owned by existing community content, so a rebuild must accept existing files unchanged (see [Data and persistence](../../../../SYSTEM-REQUIREMENTS.md#5-data-and-persistence)).

## Reading order

1. [`TuneupFileFormat.h`](TuneupFileFormat.h.md) — the document records
2. [`TuneupFileFormat.cpp`](TuneupFileFormat.cpp.md) — parsing and writing rules
