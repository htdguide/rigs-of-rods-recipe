# source/main/utils/Utils.cpp

> Implements the helpers declared in `Utils.h`.

**Needs** — [`Utils.h`](Utils.h.md) · [`system/CVar.h`](../system/CVar.h.md) · [`network/RoRnet.h`](../network/RoRnet.h.md) · [`../../version_info/RoRVersion.h`](../../version_info/RoRVersion.h.md) · [`SHA1.h`](SHA1.h.md)
**Used by** — callers of [`Utils.h`](Utils.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Small, independent helpers. Only the ones whose output is persisted or compared need exact behaviour; the rest are conveniences.

## State

Stateless.

## `sha1sum(bytes)` / `Sha1Hash(text)`

**Contract** — SHA-1 of the input as 40 **uppercase** hex characters (see [`SHA1.cpp`](SHA1.cpp.md)). Used for mod identity and cache keys, so the case matters for matching stored values.

## `HashData(bytes)`

**Contract** — lowercase hex of a fast non-cryptographic 32-bit hash (the rendering engine's `FastHash`). Used for quick change detection only.

## `SanitizeUtf8String` / `SanitizeUtf8CString` / `tryConvertUTF`

**Contract** — copy the input, replacing every invalid UTF-8 sequence with U+FFFD. Applied to all text arriving from files and the network before display.

## `Utf8ToWideChar`

**Contract** — UTF-8 → UTF-16 for Windows APIs.

## `formatBytes(n)`

**Contract** — `"%.2f <unit>"` with base-1024 units B, KB, MB, GB, TB, EB, ZB, YB (note: PB is missing from the unit list, so values ≥ 1024⁵ are mislabelled — an original bug; a rebuild should include PB).

## `getTimeStamp`

**Contract** — wall-clock seconds since the epoch.

## `getVersionString(multiline)`

**Contract** — "Rigs of Rods version X, protocol version RoRnet_2.45, build time D, T" on one line, or the same as four lines.

## `Round(value, ndigits)`

**Contract** — round half away from zero to `ndigits` decimals.

## `JoinStrVec(tokens, delim)`

**Contract** — join with delimiter; result bounded to 500 bytes (truncated beyond).

## `IsDistanceWithin(a, b, max)`

**Contract** — `|a − b|² ≤ max²` (avoids a square root).

## `PrintMeshInfo(title, mesh)`

**Contract** — multi-line diagnostic listing of a mesh's shared and per-submesh vertex declarations (binding, offset, type, semantic, size for each element). Debug output only.

## `CvarAddFileToList` / `CvarRemoveFileFromList`

**Contract** — treat a text setting as a comma-separated set of filenames: add appends only if absent; remove deletes the first match. Used for `app_custom_scripts` and `app_recent_scripts`.

## `SplitBundleQualifiedFilename(q)`

**Contract** — `"bundle.zip:path/file.as"` → (`bundle.zip`, `path/file.as`) splitting at the **first** colon; without a colon, bundle is empty and the whole input is the filename.

**Notes** — splitting at the first colon means Windows drive letters (`C:\…`) are misread as bundle names; callers only pass resource-relative names, never absolute OS paths.
