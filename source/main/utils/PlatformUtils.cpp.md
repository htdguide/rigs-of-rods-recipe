# source/main/utils/PlatformUtils.cpp

> Windows and POSIX implementations of the path helpers.

**Needs** — [`PlatformUtils.h`](PlatformUtils.h.md) · [`Application.h`](../Application.h.md) · [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine) (file-time query via its archive layer)
**Used by** — callers of [`PlatformUtils.h`](PlatformUtils.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Platform glue. Nothing here is algorithmic except `GetParentDirectory`; a rebuild uses its standard library's path API for the rest.

## State

Stateless.

## Platform behaviour

| Function | Windows | POSIX |
|---|---|---|
| user home | the **Documents** folder (`CSIDL_PERSONAL`), not the profile root | `$HOME` |
| executable path | module file name of the process | `/proc/self/exe` link target (Linux) |
| file/folder exists | attribute query, distinguishing file from directory | `stat` succeeds — **does not distinguish** file from directory |
| create folder | create if absent | `mkdir` mode 0775-ish (rwx user+group, r-x others) |
| open URL | shell "open" | `xdg-open <url>` |

**Notes** — the POSIX existence checks not distinguishing files from folders is an original quirk; a rebuild should distinguish them.

## `GetParentDirectory`

**Contract** — strips trailing separators, then the last path component, then any separators before it. Returns empty text if nothing remains.

```text
FUNCTION parent_directory(p)
  n = length(p)
  WHILE n > 0 AND p[n-1] == SLASH: n -= 1      # "a/b/" -> "a/b"
  WHILE n > 0 AND p[n-1] != SLASH: n -= 1      # "a/b"  -> "a/"
  WHILE n > 0 AND p[n-1] == SLASH: n -= 1      # "a/"   -> "a"
  RETURN p[0..n]                                # "" if n == 0
```

## `GetFileLastModifiedTime`

**Contract** — modification time of a file, obtained through the rendering engine's file-system archive (keeps one code path for loose files and archives). Used by the mod cache to detect changed mods.
