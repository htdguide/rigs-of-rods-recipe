# source/main/gui/panels/GUI_RepositorySelector.cpp

> Repository web API client, list filtering and views, one-at-a-time downloads installed into the mods folder.

**Needs** — [`GUI_RepositorySelector.h`](GUI_RepositorySelector.h.md) · [`Application.h`](../../Application.h.md) · [`utils/bbcode/BBDocument.h`](../../utils/bbcode/BBDocument.h.md) · [`GameContext.h`](../../GameContext.h.md) · [`AppContext.h`](../../AppContext.h.md) · [`system/Console.h`](../../system/Console.h.md) · [`resources/ContentManager.h`](../../resources/ContentManager.h.md) · [`GUIManager.h`](../GUIManager.h.md) · [`GUIUtils.h`](../GUIUtils.h.md) · [`utils/Language.h`](../../utils/Language.h.md) · [`utils/PlatformUtils.h`](../../utils/PlatformUtils.h.md) · [`RoRVersion.h`](../../../../source/version_info/RoRVersion.h.md) · [Seam: Immediate-mode GUI](../../../../SYSTEM-REQUIREMENTS.md#seam-immediate-mode-gui)
**Used by** — callers of [`GUI_RepositorySelector.h`](GUI_RepositorySelector.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`GUI_RepositorySelector.h`](GUI_RepositorySelector.h.md). Network calls run on worker threads and report back through game messages ([Seam: HTTP client](../../../../SYSTEM-REQUIREMENTS.md#seam-http-client), [Seam: JSON](../../../../SYSTEM-REQUIREMENTS.md#seam-json)). All requests use IPv4, gzip, the game user agent and (Windows) the OS CA store.

## State

See [`GUI_RepositorySelector.h`](GUI_RepositorySelector.h.md).

## Web API (base = setting `remote_query_url`)

| Request | Response fields used |
|---|---|
| `GET /resources` | `resources[]`: `title`, `tag_line`, `resource_id`, `download_count`, `last_update`, `resource_category_id`, `icon_url`, `rating_avg`, `rating_count`, `version`, `custom_fields.authors`, `view_url`, `resource_date`, `view_count` |
| `GET /resource-categories` (after the list) | `categories[]`: `title`, `resource_category_id`, `resource_count`, `description`, `display_order` |
| `GET /resources/<id>` | `resource.current_files[]`: `id`, `filename`, `size`; `resource.description` (BBCode); `resource.resource_id` |
| file download | `https://forum.rigsofrods.org/resources/<resource>/download?file=<file id>` |
| attachment image | `https://forum.rigsofrods.org/attachments/<id>` |
| thumbnail | the item's `icon_url` |

Failures post REFRESH_REPOLIST_FAILURE with title, transport and HTTP codes; malformed JSON posts "Received malformed data" (text payload — the same mismatch as in the server list). Success posts the collection.

## List view

Category combo ("All" = id 1), search box (case-insensitive substring of title or authors), sort (initially by last update, newest first), view mode:
- **List** — thumbnail (96 px), title, tag line, authors, version, downloads, rating stars, last update.
- **Compact** — tiles, 3 per row (2 at ≤ 1280 px wide).
- **Basic** — table: title, authors, version, downloads, rating.

Thumbnails download lazily when first drawn (cached as `<thumbnails>/<resource id>.png`); missing icon → fallback image.

## Resource view

Title, thumbnail, link to the web page, BBCode-rendered description (inline attachment images are downloaded in bulk on first display to `<repo attachments>/<id>.<ext>`; clicking one opens a gallery view), info (authors, version, downloads, views, rating, dates). Right column: files newest first, each with size and a button — *Install*, *Reinstall*, or *Update* when the resource was updated after the local file's time — and *Delete* when installed (posts delete-bundle). Buttons hide while `<file>.part` exists (download in progress). Install status is resolved once per file through the content cache (a file with that name in any mod folder).

## Installing

Each request gets an id and is queued; only one download runs at a time. The worker streams to `<destination>.part` (destination = `<user dir>/mods/<file name>`, or the existing file's path when updating), reporting progress (percent + "Downloading … MB" text) as messages. On success the main thread deletes any old file, renames `.part` into place, adds the zip to the content cache, marks the file installed, and starts the next queued download. A footer shows the queue length and progress.

## Images

Image downloads run on the renderer's work queue (one channel); results come back as REPOIMAGE success/failure messages and are loaded into textures on the main thread. An already downloaded file is reused without a request.
