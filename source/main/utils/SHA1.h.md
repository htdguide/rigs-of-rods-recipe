# source/main/utils/SHA1.h

> Incremental SHA-1 hasher.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`SHA1.cpp`](SHA1.cpp.md) · [`Utils.cpp`](Utils.cpp.md)
**Tier floor** — T4 in concept

## Purpose

Standard FIPS 180-1 SHA-1, used for mod identity and cache keys (never for security). Any library implementation is a drop-in replacement; the only non-standard detail is the text format of the digest. Implementation: [`SHA1.cpp`](SHA1.cpp.md).

## State

```text
RECORD Sha1
  state  : 5 × int (32-bit)
  count  : 64-bit bit counter
  buffer : bytes (64)
  digest : bytes (20)
```

## `Reset` · `UpdateHash(bytes)` · `Final()` · `GetHash()` · `ReportHash()`

**Contract** — reset to initial constants; feed any number of chunks; finalize (pads and produces the 20-byte digest); copy the raw digest; or return it as text — see the `.cpp` twin.
