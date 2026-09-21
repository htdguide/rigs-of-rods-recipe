# source/main/utils/GenericFileFormat.cpp

> Character-level tokeniser and serializer for the generic document format.

**Needs** — [`GenericFileFormat.h`](GenericFileFormat.h.md) · [`Application.h`](../Application.h.md) · [`system/Console.h`](../system/Console.h.md)
**Used by** — callers of [`GenericFileFormat.h`](GenericFileFormat.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Defines the lexical grammar exactly. It is a one-pass state machine over bytes (no look-ahead), so malformed input degrades token by token instead of failing the file; every discarded fragment produces a console warning `"<file>, line L, pos P: …"`.

## State

```text
RECORD Tokenizer
  doc, options, stream_name
  pending   : bytes          # characters of the token being built
  kind      : PartialKind    # what the pending token looks like so far
  line, pos : int            # for warnings only
  title_found : bool         # for FIRST_LINE_IS_TITLE

ENUM PartialKind =
  NONE | COMMENT(';' | '//' | '#') | STRING_QUOTED | STRING_NAKED | STRING_NAKED_CAPTURING_SPACES
  | TITLE | MINUS_STUB | INTEGER | DECIMAL | SCI_STUB | SCI_STUB_MINUS | SCIENTIFIC
  | KEYWORD | KEYWORD_BRACED | BOOL_TRUE | BOOL_FALSE | GARBAGE
```

"At line start" below means: no token yet, or the last emitted token is LINEBREAK.

## Lexical rules

**Contract** — the grammar, as the state machine enforces it:

- **Always ignored:** `\r`.
- **Separators:** space, tab, comma; also `:` with ALLOW_SEPARATOR_COLON and `=` with ALLOW_SEPARATOR_EQUALS. When disabled, `:`, `=`, `/`, `#` and `[` start a naked string (if allowed) or garbage.
- **Line break:** `\n` finishes any pending token (quoted strings included, with a warning "quoted string interrupted by newline") and emits LINEBREAK.
- **Comments:** `;` anywhere starts a comment to end of line; `//` and `#` do the same when enabled. For `//`, any run of leading slashes is dropped. The comment text excludes the marker.
- **Quoted strings:** `"…"`, may contain spaces, tabs, commas, `:` and `=`; cannot span lines.
- **Numbers:** start with a digit, `-` or `.`. Grammar: `-?digits` → INT; with a `.` → FLOAT; `e`/`E` followed by optional `-` and digits → FLOAT in scientific notation. A lone `-` followed by a separator is a naked STRING if naked strings are allowed. Any other character inside a number turns it into a naked string (or garbage). Values are parsed locale-independently.
- **Booleans:** exactly lowercase `true` / `false` followed by a separator. A token that starts like one but diverges (`tr_`, `falsey`) becomes a KEYWORD if at line start, else a naked string, else garbage.
- **Keywords:** only at line start, first char a letter, then letters, digits, `_`. With ALLOW_BRACED_KEYWORDS, `[` … `]` at line start is a keyword including the brackets. A non-alphanumeric character demotes it to a naked string (or garbage). With naked strings allowed, `(` inside a keyword turns it into a naked string (capturing spaces if that option is on).
- **Naked strings** (only with ALLOW_NAKED_STRINGS): end at space, tab, comma, or an enabled `:`/`=` separator; a `"` inside one is garbage; with PARENTHESES_CAPTURE_SPACES, spaces between `(` and the matching `)` are kept (e.g. `Engine (V8 turbo)`).
- **Title line** (FIRST_LINE_IS_TITLE): the first token that starts a line and is not a `;` or `//` comment makes the whole rest of that line one STRING.
- **Garbage** accumulates until a separator or line break and is then discarded with a warning.
- **End of input:** a pending quoted string, capturing naked string or title is emitted as STRING, a braced keyword as KEYWORD; anything else is finished as if a space followed. A final LINEBREAK is appended if missing.

## `saveToDataStream`

**Contract** — reproduces a canonical text form, not the original bytes:

```text
FUNCTION save(doc, out)
  sep = ""
  FOR EACH tok IN doc.tokens
    CASE tok.type OF
      LINEBREAK: write platform EOL ("\r\n" on Windows, "\n" elsewhere); sep = ""
      COMMENT:   write ";" + text                       # no separator before comments
      KEYWORD:   write text; sep = " "
      STRING:    write sep + text; sep = ", "           # NOT re-quoted
      FLOAT:     write sep + shortest %g form; sep = ", "
      INT:       write sep + integer; sep = ", "
      BOOL:      write sep + "true"/"false"; sep = ", "
```

**Notes** — strings are written without quotes, so a document containing strings with spaces only round-trips when read back with naked strings and space-capturing options that make them unambiguous. A rebuild that wants lossless round-trips should quote strings containing separators; the original does not.

## Cursor helpers

`seekNextLine`, `countLineArgs`, `appendTokens`, `insertToken`, `eraseToken`, `setStringData`, `setFloatData` — behaviour as stated in the header twin.
