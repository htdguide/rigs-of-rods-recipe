# source/main/utils/SHA1.cpp

> Standard SHA-1 with an uppercase-hex report.

**Needs** — [`SHA1.h`](SHA1.h.md)
**Used by** — callers of [`SHA1.h`](SHA1.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

A textbook SHA-1 (80 rounds, big-endian message schedule, `0x80` padding to 56 mod 64 followed by the 64-bit big-endian bit length). Use your language's library implementation.

## State

See the header twin.

## `ReportHash`

**Contract** — the 20-byte digest as **40 uppercase hexadecimal characters**, no separators. Values stored in the mod cache and compared across runs use this format.

**Notes** — `Final` wipes internal buffers after producing the digest (a memory-hygiene habit of the original library, not a requirement).
