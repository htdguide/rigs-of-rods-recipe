# source/main/utils/bbcode/BBDocument.h

> Forgiving BBCode parser that builds a node tree from forum-style markup.

**Needs** — nothing
**Used by** — [`gui/panels/GUI_RepositorySelector.cpp`](../../gui/panels/GUI_RepositorySelector.cpp.md) · [`gui/panels/GUI_RepositorySelector.h`](../../gui/panels/GUI_RepositorySelector.h.md) · [`BBDocument.cpp`](BBDocument.cpp.md)
**Tier floor** — T3

## Purpose

Mod descriptions from the online repository are written in BBCode (`[b]`, `[url=…]`, `[img]`, `[QUOTE user=Bob]`). The repository browser parses them into a tree and renders it with the immediate-mode GUI. Adopted from a small third-party library (bbcpp, MIT) with two RoR changes noted below. Any BBCode parser that produces the same tree is a substitute.

## State

```text
ENUM NodeType = DOCUMENT | ELEMENT | TEXT
ENUM ElementType = SIMPLE | VALUE | PARAMETER | CLOSING

RECORD Node
  name     : text          # tag name, or the text itself for TEXT nodes
  type     : NodeType
  parent   : weak reference to Node
  children : list<Node>

RECORD Element EXTENDS Node
  element_type : ElementType
  parameters   : map<text, text>   # insert-only: a repeated key keeps its first value

RECORD Document EXTENDS Node
  open : stack<Element>            # currently open elements while parsing
```

## `Document.load(text)`

**Contract** — parses the whole string; never fails. Malformed markup becomes text.

```text
FUNCTION load(text)
  i = 0
  WHILE i < len(text)
    IF text[i] == '['
      j = parse_element(i)
      IF j == i: i = parse_text(i)       # not an element after all
      ELSE i = j
    ELSE
      i = parse_text(i)

FUNCTION parse_text(i)          # up to (not including) the next '['; if the '[' is at i, take the rest
  emit_text(text[i .. next '[' or end])

FUNCTION parse_element(i)       # text[i] == '['
  closing = text[i+1] == '/'
  name = longest run of [A-Za-z0-9*] after '[' or '[/'
  IF name empty: emit_text("["); RETURN position after name
  IF input ended: emit_text(rest); RETURN end
  CASE next char OF
    ']'      : IF closing: close_element(name) ELSE open_element(name, SIMPLE); RETURN after ']'
    '=' or ' ': pairs = parse_key_value_pairs(...)       # "[url=X]" -> {url: X}; "[quote user=Bob id=3]"
                IF pairs empty: emit_text(consumed text); RETURN
                open_element(name, PARAMETER, pairs); RETURN after ']'
    else     : emit_text(consumed text); RETURN
```

Key/value rules: keys are alphanumeric runs ending in `=`; values run to the closing `]`. **RoR additions:** a value may be wrapped in double quotes (then it ends at the closing quote), and an unquoted value may contain balanced `[`…`]` (so `[url=https://x/[1]]` works). Any malformed pair discards all pairs and the tag becomes text.

## Tree building

**Contract** —
- `emit_text(s)` appends to the previous TEXT sibling if the last child of the current parent is text, otherwise adds a new TEXT child. The current parent is the top of `open`, or the document.
- `open_element` adds the element to the current parent and pushes it.
- `close_element` adds a CLOSING marker node to the current parent and pops **whatever is on top**, without checking the name — mismatched tags therefore close the innermost open element.

## Utilities

**Contract** — `is_digit`, `is_alpha` (ASCII letters only), `is_alnum`, `is_space`; typed down-casts of nodes that either fail softly (return empty) or raise.
