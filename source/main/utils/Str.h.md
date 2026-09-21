# source/main/utils/Str.h

> Fixed-capacity, stack-allocated text buffer with append operators.

**Needs** — nothing
**Used by** — [`Application.h`](../Application.h.md)
**Tier floor** — T4 in concept

## Purpose

A bounded string builder that never allocates and silently truncates at its capacity `L` (including terminator). Used where the original wanted zero heap traffic (path building at startup, legacy formatters) and where text has a hard wire-format limit.

## State

```text
RECORD Str<L>
  buffer : bytes (L)     # always NUL-terminated; unused tail zero-filled
```

## Append / assign / compare

**Contract** — appends text, a char, an int (`%d`), a size, or a float (`%f`, six decimals); assignment clears then appends; appends that exceed `L − 1` bytes are truncated, never overflowing. Comparison is byte-wise over at most `L` bytes.

**Notes** — in a managed language use the standard string builder; keep an explicit truncation wherever the text is later copied into a fixed-size wire or file field (e.g. RoRnet usernames, 40 bytes).
