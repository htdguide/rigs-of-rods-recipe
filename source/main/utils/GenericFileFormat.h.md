# source/main/utils/GenericFileFormat.h

> A token-stream document for RoR's family of line-oriented text formats, plus a cursor for reading and editing it.

**Needs** — [`memory/RefCountingObject.h`](memory/RefCountingObject.h.md) · [`memory/RefCountingObjectPtr.h`](memory/RefCountingObjectPtr.h.md) · [`BitFlags.h`](BitFlags.h.md) · [`Application.h`](../Application.h.md)
**Used by** — [`gui/DashBoardManager.cpp`](../gui/DashBoardManager.cpp.md) · [`resources/CacheSystem.cpp`](../resources/CacheSystem.cpp.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.cpp`](../resources/addonpart_fileformat/AddonPartFileFormat.cpp.md) · [`resources/addonpart_fileformat/AddonPartFileFormat.h`](../resources/addonpart_fileformat/AddonPartFileFormat.h.md) · [`scripting/bindings/GenericFileFormatAngelscript.cpp`](../scripting/bindings/GenericFileFormatAngelscript.cpp.md) · [`GenericFileFormat.cpp`](GenericFileFormat.cpp.md)
**Tier floor** — T4

## Purpose

Many newer RoR formats (tune-ups, add-on parts, dashboards' side files, terrain object lists in the editor, and anything scripts want to read) share one lexical grammar. Instead of a parser per format, the file is tokenised once into a flat list of typed tokens with explicit line breaks; each format reader then walks that list with a cursor. Scripts get the same API, so mods can parse and write these files too. Tokenising rules: [`GenericFileFormat.cpp`](GenericFileFormat.cpp.md).

## State

```text
ENUM TokenType = NONE | LINEBREAK | COMMENT | STRING | FLOAT | INT | BOOL | KEYWORD

RECORD Token
  type : TokenType
  data : real (32-bit)   # numeric value; or, for COMMENT/STRING/KEYWORD, the byte offset into string_pool

RECORD GenericDocument
  tokens      : list<Token>
  string_pool : bytes     # NUL-terminated strings appended back to back

RECORD GenericDocContext           # a cursor
  doc       : GenericDocument (shared)
  token_pos : int = 0
```

Invariant after loading: the token list ends with a LINEBREAK.

**Notes** — string offsets are stored in a 32-bit float, which is exact only up to 2²⁴ (16 MiB of pooled text); the original accepts that ceiling. A rebuild should store a proper integer or a string reference.

## Parse options (bit flags)

| Bit | Option | Effect |
|---|---|---|
| 1 | ALLOW_NAKED_STRINGS | unquoted non-keyword text becomes STRING instead of garbage |
| 2 | ALLOW_SLASH_COMMENTS | `//` starts a comment |
| 3 | FIRST_LINE_IS_TITLE | the first non-empty, non-comment line is one STRING, spaces included |
| 4 | ALLOW_SEPARATOR_COLON | `:` separates tokens |
| 5 | PARENTHESES_CAPTURE_SPACES | in a naked string, spaces between `(` and `)` are kept |
| 6 | ALLOW_BRACED_KEYWORDS | `[name]` at line start is a KEYWORD (INI sections) |
| 7 | ALLOW_SEPARATOR_EQUALS | `=` separates tokens |
| 8 | ALLOW_HASH_COMMENTS | `#` starts a comment |

## `GenericDocument` I/O

**Contract** — `loadFromDataStream(stream, options)` replaces the contents by tokenising the stream; `saveToDataStream(stream)` writes it back (rules in the `.cpp` twin); `loadFromResource(name, group, options)` / `saveToResource(name, group)` do the same through the resource system and return false with a console error on I/O failure.

## `GenericDocContext` — reading

**Contract** — every accessor takes an `offset` relative to `token_pos` (default 0) and is safe past the end:
- `moveNext()` advances one token and returns whether the end was reached.
- `getPos()`; `endOfFile(offset)`.
- `seekNextLine()` skips to the start of the next line that is neither empty nor a comment; returns end-of-file.
- `countLineArgs()` counts tokens from the cursor to the next LINEBREAK.
- `tokenType(offset)` — NONE past the end.
- `isTok{String,Float,Int,Bool,Keyword,Comment,LineBreak,Numeric}(offset)` — Numeric is Int or Float.
- `getTok{String,Float,Int,Numeric,Bool,Keyword,Comment}(offset)` — the typed value; reading the wrong type is a programming error (asserted). Past the end, strings are empty and numbers 0.

## `GenericDocContext` — editing

**Contract** —
- `appendTokens(n)` adds `n` NONE tokens at the end and moves the cursor to the first of them.
- `insertToken(offset)` / `eraseToken(offset)` insert a NONE / remove a token; false if `offset` is past the end.
- `setTok{String,Float,Int,Bool,Keyword,Comment,LineBreak}(offset, value)` overwrite a token's type and value; false past the end. Strings are appended to the pool (old text is left as garbage — the pool only grows).
- `appendTok…(value)` = `appendTokens(1)` + `setTok…(0, value)`.

The order of these methods is mirrored in the script binding and its documentation.
