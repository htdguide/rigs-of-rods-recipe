# source/main/resources/otc_fileformat/OTCFileFormat.h

> The `.otc` heightmap-terrain configuration: world size, paging grid, rendering options and per-page texture layers.

**Needs** — [Seam: 3D rendering engine](../../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine) (data stream, vector, colour)
**Used by** — [`OTCFileFormat.cpp`](OTCFileFormat.cpp.md) · [`terrain/TerrainGeometryManager.cpp`](../../terrain/TerrainGeometryManager.cpp.md) · [`terrain/TerrainGeometryManager.h`](../../terrain/TerrainGeometryManager.h.md)
**Tier floor** — T4

## Purpose

Describes the heightmap terrain referenced by a `.terrn2`: one master file plus one page file per terrain page. Coordinates: X right/left, Y up, Z front/back. Parsing: [`OTCFileFormat.cpp`](OTCFileFormat.cpp.md).

## State

```text
RECORD OTCLayer
  world_size               : real       # texture tiling size in metres
  diffusespecular_filename : text
  normalheight_filename    : text
  blendmap_filename        : text
  blend_mode               : char = 'R' # which channel of the blend map ('R','G','B','A')
  alpha                    : real = 82  # see Notes

RECORD OTCPage
  pageconf_filename  : text
  heightmap_filename : text
  num_layers         : int
  pos_x, pos_z       : int
  is_heightmap_raw   : bool             # true if the heightmap name contains ".raw"
  raw_flip_x, raw_flip_y : bool
  raw_size, raw_bpp  : int              # RAW heightmaps: edge length and bytes per sample
  layers             : list<OTCLayer>

RECORD OTCDocument
  page_filename_format   : text        # with {X} and {Z} placeholders
  cache_filename_base    : text
  pages                  : list<OTCPage>
  origin_pos             : (x, y, z)   # (world_size_x/2, 0, world_size_z/2)
  world_size_x, world_size_y (height scale), world_size_z, world_size (= max(x, z)) : int
  page_size              : int         # heightmap samples per page edge (2^n + 1)
  pages_max_x, pages_max_z : int       # highest page index (inclusive)
  max_pixel_error, batch_size_min, batch_size_max,
  layer_blendmap_size, composite_map_size, composite_map_distance,
  skirt_size, lightmap_size : int
  lightmap_enabled, norm_map_enabled, spec_map_enabled, parallax_enabled,
  global_colormap_enabled, recv_dyn_shadows_depth, blendmap_dbg_enabled,
  disable_cache, is_flat : bool
```

**Notes** — the layer `alpha` default is the numeric value of the character `'R'` (82.0), an old typo preserved for compatibility; content that omits alpha therefore gets 82.

## `OTCParser`

**Contract** — `LoadMasterConfig(stream, filename)`, `LoadPageConfig(stream, page, filename)`, `GetDefinition()`. See `.cpp` twin.
