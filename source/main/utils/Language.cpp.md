# source/main/utils/Language.cpp

> Discovers installed translations and loads the one matching the configured language.

**Needs** — [`Language.h`](Language.h.md) · [`Application.h`](../Application.h.md) · [`PlatformUtils.h`](PlatformUtils.h.md)
**Used by** — callers of [`Language.h`](Language.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Translations are gettext catalogs at `<process_dir>/languages/<code>/ror.mo`. The catalog's own header (the translation of the empty string) names the language.

## State

See header twin; the active catalog lives in the translation library's singleton.

## `setup`

**Contract** — must run after settings are loaded and the rendering engine exists (it uses the engine's file listing).

```text
FUNCTION setup()
  languages = [("English", "en")]
  FOR EACH subdirectory d OF process_dir/"languages"
    IF catalog d/"ror.mo" loads
      languages.append(extract_lang(catalog.lookup("")))
  SORT languages[1..] by display name      # English stays first
  clear the catalog
  code = first two characters of app_language
  IF catalog process_dir/"languages"/code/"ror.mo" loads
    app_language = extract_lang(header).code       # normalises e.g. "de_DE" -> header's code
    LOG loaded
  ELSE
    LOG error; translations stay off (English keys shown)

FUNCTION extract_lang(header_text) -> (display_name, code)
  FOR EACH line IN header_text split by newline
    tokens = line split by whitespace
    IF tokens[0] == "Language-Team:": display_name = tokens[1]
    IF tokens[0] == "Language:":      code = tokens[1]
```

**Notes** — only the first word of `Language-Team` is kept (e.g. "German" from "German <team@...>"); the directory name is truncated to two letters, so region variants share a directory.
