# source/main/utils/bbcode — BBCode parsing

Part of chapter 5. A vendored, lightly modified copy of the bbcpp library (MIT), used only by the online repository browser to render mod descriptions. Leaf: depends on nothing in RoR.

| Twin | Role |
|---|---|
| [`BBDocument.h`](BBDocument.h.md) | Grammar, node tree, parsing algorithm |
| [`BBDocument.cpp`](BBDocument.cpp.md) | Tree-building callbacks |

The original `README.txt` records provenance (upstream commit 852a02d) and that the library's HTML-rendering utilities were folded into [`GUI_RepositorySelector.cpp`](../../gui/panels/GUI_RepositorySelector.cpp.md).
