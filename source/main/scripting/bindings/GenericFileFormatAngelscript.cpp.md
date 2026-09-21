# source/main/scripting/bindings/GenericFileFormatAngelscript.cpp

> Script access to the generic tokenised text document (read, edit, save).

**Needs** — [`AngelScriptBindings.h`](AngelScriptBindings.h.md) · [`utils/GenericFileFormat.h`](../../utils/GenericFileFormat.h.md) · [Seam: Script engine](../../../../SYSTEM-REQUIREMENTS.md#seam-script-engine)
**Used by** — [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup) at startup, through [`AngelScriptBindings.h`](AngelScriptBindings.h.md)
**Tier floor** — T2

## Purpose

Registers part of the script-visible API. Names, signatures and enum values here are a compatibility contract with existing mod scripts: a rebuild keeps them even where the native side is renamed. Each function is called once from engine startup in [`ScriptEngine.cpp`](../ScriptEngine.cpp.md#startup).

## State

Stateless — registration only.

## `GenericDocumentClass`

Reference-counted, created with `GenericDocument()`. `loadFromResource(file, group, options)`, `saveToResource(file, group)`. Options enum `GenericDocumentOptions`: ALLOW_NAKED_STRINGS, ALLOW_SLASH_COMMENTS, FIRST_LINE_IS_TITLE, ALLOW_SEPARATOR_COLON, PARENTHESES_CAPTURE_SPACES, ALLOW_BRACED_KEYWORDS, ALLOW_SEPARATOR_EQUALS, ALLOW_HASH_COMMENTS.

## `GenericDocContextClass`

A cursor over a document, created with `GenericDocContext(doc)`. Navigation: `moveNext`, `getPos`, `seekNextLine`, `countLineArgs`, `endOfFile`, `tokenType(offset)`. Reading: `getTok{String,Float,Int,Bool,Keyword,Comment}(offset)`, `isTok{String,Float,Int,Bool,Keyword,Comment,LineBreak}(offset)`. Editing: `appendTokens(n)`, `insertToken`, `eraseToken`, `appendTok…` and `setTok…` for each type. Enum `TokenType`: NONE, LINEBREAK, COMMENT, STRING, FLOAT, INT, BOOL, KEYWORD. See [`utils/GenericFileFormat`](../../utils/GenericFileFormat.h.md).
