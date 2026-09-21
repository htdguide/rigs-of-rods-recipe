# source/main/gfx/AdvancedScreen.h

> Screenshots with game metadata hidden in the pixels’ least-significant bits.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`AppContext.cpp`](../AppContext.cpp.md)
**Tier floor** — T2


## Purpose

When the user takes a "metadata" screenshot, the application ([`AppContext`](../AppContext.cpp.md)) records player name, language, truck file/name, terrain file/name, and server into the image itself, so the picture carries its context. Header-only.

## State

```text
RECORD AdvancedScreen = { window; filename; data : ordered map<key, value> (empty values skipped) }
```

## `write`

```text
pixels = copy of the window contents as 24-bit BGR
text = "RORED\n" + for each (key, value) in key order: "key:value\n"
FOR each bit of text (most significant bit first within each byte), then 40 zero bits:
  set the least-significant bit of the next pixel byte to that bit
save the image on a background thread (then free buffers)
```

Readers recover the text by collecting LSBs until five zero bytes. Capacity is limited by the 32 KiB text buffer and the image size.
