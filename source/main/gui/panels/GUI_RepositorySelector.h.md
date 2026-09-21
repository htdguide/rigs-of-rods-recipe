# source/main/gui/panels/GUI_RepositorySelector.h

> In-game browser for the online mod repository: list, search, details with BBCode descriptions and images, and queued downloads.

**Needs** — [`Application.h`](../../Application.h.md) · [`utils/bbcode/BBDocument.h`](../../utils/bbcode/BBDocument.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — [`gui/GUIManager.h`](../GUIManager.h.md) · [`GUI_RepositorySelector.cpp`](GUI_RepositorySelector.cpp.md) · [`main.cpp`](../../main.cpp.md)
**Tier floor** — T2


## Purpose

Find and install community content without leaving the game. Implementation: [`GUI_RepositorySelector.cpp`](GUI_RepositorySelector.cpp.md).

## State

```text
RECORD RepoImageDownloadRequest: thumbnail item index | attachment id + extension; resource id; URL
RECORD RepoFileInstallRequest: request id; resource id; repo file id; file name; destination path; size (display)
RECORD ResourceCategories: id, title, description, resource count, display order
RECORD ResourceItem: id, title, tag line, icon URL, authors, version, parsed BBCode description, downloads, last update,
                     category id, rating average and count, view URL, date added, views; thumbnail; download-queued flags
RECORD ResourceFiles: id, file name, size, cached install status (unknown | installed | not installed), install path
RECORD ResourcesCollection: items, categories, files
RECORD RepositorySelector
  visible; data; search (500); current category (1 = all) + labels; gallery attachment (−1 none); spinner
  sort (Last Update | Date Added | Title | Downloads | Rating | Rating Count); view (List | Compact | Basic)
  open resource (−1 = list); fallback thumbnail; loaded attachment images
  install queue + active request id + next id; image work-queue channel and handler; status messages (+ transport, HTTP)
CONSTANTS: attachment ≤160×90; title colour (1,1,.7); install button colour (.83,.655,.174)
```

## API

`SetVisible`, `IsVisible`, `Draw`, `OpenResource(id)`, `RequestInstallRepoFile(resource, file index, path)`, `QueueInstallRepoFile`, `InstallDownloadedRepoFile(result, request)`, `NotifyRepoFileUninstalled(name)`, `Refresh`, `UpdateResources(data)`, `UpdateResourceFilesAndDescription(data)`, `ShowError(info)`, `DownloadImage` (worker), `LoadDownloadedImage` (main thread), `DownloadAttachment`, `DownloadBBCodeAttachmentsRecursive`, `GetNextInstallRequestId`.
