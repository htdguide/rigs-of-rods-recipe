# source/main/terrain/OgreTerrainPSSMMaterialGenerator.h

> An adapted copy of the renderer’s standard terrain material generator with parallel-split shadow support.

**Needs** — nothing in this repository (only standard or third-party headers)
**Used by** — [`gfx/ShadowManager.h`](../gfx/ShadowManager.h.md) · [`OgreTerrainPSSMMaterialGenerator.cpp`](OgreTerrainPSSMMaterialGenerator.cpp.md) · [`TerrainGeometryManager.cpp`](TerrainGeometryManager.cpp.md)
**Tier floor** — T2


## Purpose

Third-party code (MIT-licensed, from the rendering engine's terrain component) carried in the tree with small RoR tweaks. It generates the vertex/fragment programs for terrain pages: layered diffuse/normal/specular/parallax mapping, global colour map, light map, composite map for distant pages, and receiving PSSM or depth shadows. It belongs to the [Seam: 3D rendering engine](../../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine): a rebuild uses whatever terrain shading its renderer offers. Implementation: [`OgreTerrainPSSMMaterialGenerator.cpp`](OgreTerrainPSSMMaterialGenerator.cpp.md).

## State

Profile options: layer normal/parallax/specular mapping, global colour map, light map, composite map, dynamic shadows (PSSM settings, depth, low-LOD), plus shader-helper variants (Cg, HLSL, GLSL, GLSL ES).

## API

The renderer's material-generator interface (`generate`, `generateForCompositeMap`, `updateParams`, `requestOptions`, maximum layers) and the option setters listed above.
