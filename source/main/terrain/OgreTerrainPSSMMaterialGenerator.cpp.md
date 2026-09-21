# source/main/terrain/OgreTerrainPSSMMaterialGenerator.cpp

> Shader source generation for the terrain material (third-party, adapted).

**Needs** — [`OgreTerrainPSSMMaterialGenerator.h`](OgreTerrainPSSMMaterialGenerator.h.md)
**Used by** — callers of [`OgreTerrainPSSMMaterialGenerator.h`](OgreTerrainPSSMMaterialGenerator.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

Emits shader source text per page from the enabled options (layer loops, blend-map sampling, shadow-split selection). Not specific to Rigs of Rods; recorded as part of the rendering seam and not specified further here.

## State

See [`OgreTerrainPSSMMaterialGenerator.h`](OgreTerrainPSSMMaterialGenerator.h.md).

## Contract

Given a terrain page and the active profile, returns a material with one technique per supported shading language, parameters updated each frame from the page and the shadow camera setup. The observable requirements a rebuild must keep: up to the page's layer count of blended texture layers (diffuse+specular, normal+height), optional normal/parallax/specular mapping, a global colour map, light-map and composite-map support for distant pages, and receiving shadows from the main light.
