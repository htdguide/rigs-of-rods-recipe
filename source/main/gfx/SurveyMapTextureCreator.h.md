# source/main/gfx/SurveyMapTextureCreator.h

> Renders a top-down orthographic image of the terrain for the survey (overview) map.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`SurveyMapTextureCreator.cpp`](SurveyMapTextureCreator.cpp.md) · [`gui/panels/GUI_SurveyMap.cpp`](../gui/panels/GUI_SurveyMap.cpp.md) · [`gui/panels/GUI_SurveyMap.h`](../gui/panels/GUI_SurveyMap.h.md)
**Tier floor** — T2


## Purpose

Called once per terrain (before actors spawn) to bake the map background. Implementation: [`SurveyMapTextureCreator.cpp`](SurveyMapTextureCreator.cpp.md).

## State

```text
RECORD SurveyMapTextureCreator = { camera height = clamp(terrain max height + 100, 150, 2500); texture name "MapRttTex-n"; camera; texture; render target }
```

## API

`init(resolution, fsaa) → ok`, `update(centre xz, size xz)`, `convertTextureToStatic(name, group) → texture`.
