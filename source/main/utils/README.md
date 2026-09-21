# source/main/utils — shared utilities

Chapter 5 in build order. Depends on the hub ([`Application.h`](../Application.h.md)) and, for a few helpers, on the console and settings of chapter 6; everything above uses it.

Themes:
- **Text and formats** — [`GenericFileFormat`](GenericFileFormat.h.md) is the lexer behind all newer RoR text formats; [`ConfigFile`](ConfigFile.h.md) / [`ImprovedConfigFile`](ImprovedConfigFile.h.md) handle INI-style files; [`Language`](Language.h.md) is translation; [`Str`](Str.h.md) is a bounded string; [`bbcode/`](bbcode/README.md) parses forum markup.
- **Input** — [`InputEngine`](InputEngine.h.md) owns keyboard/mouse/joysticks and maps them to ~320 named actions through `.map` files; [`ForceFeedback`](ForceFeedback.h.md) drives wheels.
- **Platform** — [`PlatformUtils`](PlatformUtils.h.md), [`ErrorUtils`](ErrorUtils.h.md).
- **Misc** — hashing ([`SHA1`](SHA1.h.md)), [`Utils`](Utils.h.md), [`BitFlags`](BitFlags.h.md), [`Vec3`](Vec3.h.md), a thread hand-off list ([`InterThreadStoreVector`](InterThreadStoreVector.h.md)), mesh placement with LODs ([`MeshObject`](MeshObject.h.md)), text-into-texture ([`WriteTextToTexture`](WriteTextToTexture.h.md)).

| Twin | Role |
|---|---|
| [`BitFlags.h`](BitFlags.h.md) | 1-based flag bits (wire-format relevant) |
| [`Str.h`](Str.h.md) | Fixed-capacity string builder |
| [`Vec3.h`](Vec3.h.md) | Value-type 3D vector for hot loops |
| [`InterThreadStoreVector.h`](InterThreadStoreVector.h.md) | Push/drain-all list between threads |
| [`ErrorUtils.h`](ErrorUtils.h.md) · [`.cpp`](ErrorUtils.cpp.md) | OS message boxes before the GUI exists |
| [`PlatformUtils.h`](PlatformUtils.h.md) · [`.cpp`](PlatformUtils.cpp.md) | UTF-8 paths, home/exe dirs, parent dir |
| [`Utils.h`](Utils.h.md) · [`.cpp`](Utils.cpp.md) | Hash/UTF-8/format helpers, world→screen projection |
| [`SHA1.h`](SHA1.h.md) · [`.cpp`](SHA1.cpp.md) | SHA-1, uppercase hex |
| [`ConfigFile.h`](ConfigFile.h.md) · [`.cpp`](ConfigFile.cpp.md) | INI reader with typed getters |
| [`ImprovedConfigFile.h`](ImprovedConfigFile.h.md) | INI read/write for script storage |
| [`Language.h`](Language.h.md) · [`.cpp`](Language.cpp.md) | gettext catalogs |
| [`MeshObject.h`](MeshObject.h.md) · [`.cpp`](MeshObject.cpp.md) | Mesh entity with `_lodN` / `_clod_D` discovery |
| [`WriteTextToTexture.h`](WriteTextToTexture.h.md) · [`.cpp`](WriteTextToTexture.cpp.md) | CPU text rasteriser |
| [`ForceFeedback.h`](ForceFeedback.h.md) · [`.cpp`](ForceFeedback.cpp.md) | Wheel force law |
| [`GenericFileFormat.h`](GenericFileFormat.h.md) · [`.cpp`](GenericFileFormat.cpp.md) | Token document + cursor; lexical grammar |
| [`InputEngine.h`](InputEngine.h.md) · [`.cpp`](InputEngine.cpp.md) | Actions, `.map` grammar, axis shaping, default bindings |
