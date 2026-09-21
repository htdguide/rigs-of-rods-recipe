# source/main/plugins.cfg.in

> Build-time template listing the rendering-engine plugins to load at startup.

**Needs** — [Seam: 3D rendering engine](../../SYSTEM-REQUIREMENTS.md#seam-3d-rendering-engine)
**Used by** — the rendering engine at startup, after the build configures it into `plugins.cfg`
**Tier floor** — T4

## Purpose

Data twin. Produces `plugins.cfg` (and an identical `plugins_d.cfg` for debug builds) next to the executable. Format: `key=value` lines, `#` comments; `PluginFolder=<dir>` once, then one `Plugin=<name>` per plugin.

## State

Plugins requested, and why each matters to a rebuild:

| Plugin | Needed for |
|---|---|
| Image codec (FreeImage) | loading PNG/JPG/DDS textures, writing screenshots |
| One renderer per platform — Direct3D 9 on Windows, OpenGL elsewhere (D3D11 and GL3+ are listed but commented out) | drawing |
| Particle effects | exhaust, dust, fire, custom particles |
| Octree scene manager | spatial culling of terrain objects |
| Cg shader program manager | legacy shaders in content (terrain PSSM, water) |
| Caelum (Windows always; Linux when installed) | the optional dynamic sky |

**Notes** — the build comments out renderers not available on the target by substituting `# ` before the line. The plugin set is the practical "must provide" list for the rendering seam: image codecs, one GPU backend, particles, a spatial scene manager, and a way to run content-supplied shaders.
