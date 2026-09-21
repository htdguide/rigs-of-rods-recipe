# source/main/utils/PlatformUtils.h

> UTF-8 path helpers: existence checks, directory creation, home/executable paths, modification time, opening URLs.

**Needs** — nothing
**Used by** — [`AppContext.cpp`](../AppContext.cpp.md) · [`gfx/GfxWater.cpp`](../gfx/GfxWater.cpp.md) · [`gui/GUIManager.cpp`](../gui/GUIManager.cpp.md) · [`gui/GUIUtils.cpp`](../gui/GUIUtils.cpp.md) · [`gui/panels/GUI_GameMainMenu.cpp`](../gui/panels/GUI_GameMainMenu.cpp.md) · [`gui/panels/GUI_RepositorySelector.cpp`](../gui/panels/GUI_RepositorySelector.cpp.md) · [`gui/panels/GUI_TextureToolWindow.cpp`](../gui/panels/GUI_TextureToolWindow.cpp.md) · [`gui/panels/GUI_TopMenubar.cpp`](../gui/panels/GUI_TopMenubar.cpp.md) · [`main.cpp`](../main.cpp.md) · [`physics/Savegame.cpp`](../physics/Savegame.cpp.md) · [`physics/collision/Collisions.cpp`](../physics/collision/Collisions.cpp.md) · [`physics/flex/FlexFactory.cpp`](../physics/flex/FlexFactory.cpp.md) · [`physics/water/Wavefield.cpp`](../physics/water/Wavefield.cpp.md) · [`resources/CacheSystem.cpp`](../resources/CacheSystem.cpp.md) · [`resources/ContentManager.cpp`](../resources/ContentManager.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md) · [`scripting/LocalStorage.cpp`](../scripting/LocalStorage.cpp.md) · [`scripting/ScriptEngine.cpp`](../scripting/ScriptEngine.cpp.md) · [`system/AppCommandLine.cpp`](../system/AppCommandLine.cpp.md) · [`system/AppConfig.cpp`](../system/AppConfig.cpp.md) · [`terrain/TerrainEditor.cpp`](../terrain/TerrainEditor.cpp.md) · [`terrain/TerrainObjectManager.cpp`](../terrain/TerrainObjectManager.cpp.md) · [`Language.cpp`](Language.cpp.md) · [`PlatformUtils.cpp`](PlatformUtils.cpp.md)
**Tier floor** — T4

## Purpose

Isolates the only OS-specific file-system calls. All paths in the program are **UTF-8 narrow strings**; conversion to the OS's native encoding happens only here. Implementation: [`PlatformUtils.cpp`](PlatformUtils.cpp.md).

## State

```text
PATH_SLASH : char    # '\' on Windows, '/' elsewhere
```

## Functions

**Contract** —
- `FileExists(path)`, `FolderExists(path)`, `CreateFolder(path)` (single level; no-op if present)
- `PathCombine(a, b)` = `a + PATH_SLASH + b` (no normalisation)
- `GetUserHomeDirectory()`, `GetExecutablePath()` — empty text on failure
- `GetParentDirectory(path)` — see the `.cpp` twin
- `GetFileLastModifiedTime(path)` — seconds since epoch
- `OpenUrlInDefaultBrowser(url)`
