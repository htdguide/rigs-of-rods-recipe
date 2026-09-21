# source/main/utils/WriteTextToTexture.cpp

> CPU text layout and blending into a texture region.

**Needs** — [`WriteTextToTexture.h`](WriteTextToTexture.h.md) · [`Application.h`](../Application.h.md) · [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine)
**Used by** — callers of [`WriteTextToTexture.h`](WriteTextToTexture.h.md) (see its Used by)
**Tier floor** — T2

## Purpose

A small monospace-ish text rasteriser working from a font atlas. Byte-per-character (no UTF-8 decoding): only single-byte glyphs are supported.

## State

A cache of scaled fonts in the engine's font manager, named `"WTTFont_<size+dpi>_<source>"`, created on first use from the reference font's TrueType source.

## `WriteToTexture`

```text
FUNCTION write_to_texture(text, dest, rect, font, colour, size, dpi, justify, wordwrap)
  clip rect to dest size
  f = cached font(size, dpi) or create it
  atlas = CPU copy of f's glyph atlas            # atlas is write-only on the GPU, so copy it
  FOR EACH character c: glyph_box[c] = atlas rectangle (skip space, tab, newline)
  char_w = widest glyph; char_h = tallest glyph
  x, y = 0, 0; at_line_start = true; line_end = rect.width
  FOR EACH character c AT index i
    CASE c OF
      ' '  : x += char_w
      '\t' : x += 3 * char_w
      '\n' : y += char_h; at_line_start = true
      else :
        IF x + glyph_box[i].w > line_end AND NOT at_line_start
          y += char_h; at_line_start = true          # wrap
        IF at_line_start
          line_w = measure_line(text, i, rect.width, wordwrap)   # words that fit
          CASE justify OF
            'c': x = (rect.w - line_w) / 2; line_end = rect.w - x
            'r': x = rect.w - line_w;       line_end = rect.w
            else: x = 0;                    line_end = line_w
          at_line_start = false
        IF y + char_h > rect.height: STOP        # out of room
        blend glyph_box[i] at (x, y): alpha = colour.a * atlas_green_channel / 255
                                       dest = dest * (1 - alpha) + colour * alpha
        x += glyph_box[i].w
```

`measure_line` sums widths word by word (a word ends at space/tab/newline, or is a single character when `wordwrap` is off), stopping before the first word that would overflow; if even the first word overflows, the line width is the full rectangle.

**Notes** — alpha is read from the atlas pixel's **second byte** (the luminance/alpha channel of the engine's font atlas format). A rebuild with a different atlas format reads its coverage channel instead.
