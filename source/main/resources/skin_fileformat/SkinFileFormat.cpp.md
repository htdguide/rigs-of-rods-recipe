# source/main/resources/skin_fileformat/SkinFileFormat.cpp

> Reads `.skin` files.

**Needs** — [`SkinFileFormat.h`](SkinFileFormat.h.md) · [`Application.h`](../../Application.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md)
**Used by** — callers of [`SkinFileFormat.h`](SkinFileFormat.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Defines the `.skin` grammar.

## State

Stateless.

## Grammar

```text
<skin name>            # first non-blank, non-comment line; trimmed
{                      # the parser skips forward to the next line containing "{"
  replacetexture  = <original>, <replacement>
  replacematerial = <original>, <replacement>
  preview         = <image file>
  description     = <text>
  authorname      = <text>
  authorid        = <int>
  guid            = <vehicle guid>        # stored trimmed, lowercased
  name            = <text>                # overrides the header name
}
<next skin name>
{ … }
```

Lines are UTF-8-sanitised. Blank lines and lines starting with `//` are skipped. Attribute lines split on tab, `=`, `,`, `;` into trimmed fields; the attribute name is case-insensitive. `replacetexture`/`replacematerial` need exactly 3 fields and `authorid` exactly 2; others need at least 2 (extra fields ignored). Unknown attributes are ignored. A closing line must be exactly `}`.

```text
FUNCTION parse_skins(stream)
  skins = []; current = none
  FOR EACH line IN stream
    IF blank OR starts with "//": CONTINUE
    IF current is none
      current = new skin named trim(line); skip to line after "{"
    ELSE IF line == "}"
      skins.append(current)
      # NOTE: current is not cleared (see Notes)
    ELSE
      apply attribute(line, current)
  IF current is set at end of file: warn "not properly closed with '}'" and append it anyway
  RETURN skins
```

**Notes** — after a `}` the original moves the finished skin into the result list and leaves the "current" slot empty *only because the move empties it*; the effect is that the next non-comment line starts a new skin. A rebuild should model this explicitly. On a read error the parse stops with a warning and the skins found so far are returned.
