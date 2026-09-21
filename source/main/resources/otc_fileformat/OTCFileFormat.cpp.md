# source/main/resources/otc_fileformat/OTCFileFormat.cpp

> Reads the `.otc` master and page files.

**Needs** — [`OTCFileFormat.h`](OTCFileFormat.h.md) · [`Application.h`](../../Application.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`physics/SimConstants.h`](../../physics/SimConstants.h.md) · [`utils/ConfigFile.h`](../../utils/ConfigFile.h.md) · [`utils/Utils.h`](../../utils/Utils.h.md)
**Used by** — callers of [`OTCFileFormat.h`](OTCFileFormat.h.md) (see its Used by)
**Tier floor** — T4

## Purpose

Defines both `.otc` grammars.

## State

The document under construction.

## `LoadMasterConfig(stream, filename)`

**Contract** — INI without sections (separators tab, `:`, `=`; whitespace not trimmed by the reader). Keys and defaults:

| Key | Default |
|---|---|
| `disableCaching` | false |
| `WorldSizeX` / `WorldSizeY` / `WorldSizeZ` | 1024 / 50 / 1024 |
| `PageSize` | 1025 |
| `PagesX` / `PagesZ` | 0 / 0 (highest index; one page = 0,0) |
| `PageFileFormat` | `<basename>-page-{X}-{Z}.otc` |
| `Flat` | false |
| `MaxPixelError` | 5 |
| `minBatchSize` / `maxBatchSize` | 33 / 65 |
| `LightmapEnabled`, `NormalMappingEnabled`, `SpecularMappingEnabled`, `ParallaxMappingEnabled`, `DebugBlendMaps`, `GlobalColourMapEnabled`, `ReceiveDynamicShadowsDepth` | false |
| `CompositeMapDistance` | 4000 |
| `LayerBlendMapSize`, `CompositeMapSize`, `LightMapSize` | 1024 |
| `SkirtSize` | 30 |
| per page `Heightmap.<x>.<z>.raw.size` | 1025 |
| per page `Heightmap.<x>.<z>.raw.bpp` | 2 |
| per page `Heightmap.<x>.<z>.flipX` / `.flipY` | false |

Then a page record is created for every `x` in `0..PagesX` and `z` in `0..PagesZ`, with the page file name from the format (placeholders replaced by the integers). The cache base name is the master file's base name. Any failure is reported as a generic exception and returns false.

## `LoadPageConfig(stream, page, filename)`

**Contract** — line-based:

```text
line 1: heightmap file name
line 2: declared layer count (int)
then per layer (skip blank lines and lines starting with ';' or '/'):
  world_size, [diffuse_specular], [normal_height], [blendmap], [blend_mode char], [alpha]
```

Fields are comma-separated and trimmed. A heightmap name containing `.raw` marks the page as RAW. If the declared layer count differs from the number of layer lines, a warning is logged and the actual count is used.
