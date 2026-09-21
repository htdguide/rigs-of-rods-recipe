# source/main/network/CurlHelpers.cpp

> HTTP GET with fixed client identity; result posted as a script-thread-status event.

**Needs** — [`Application.h`](../Application.h.md) · [`CurlHelpers.h`](CurlHelpers.h.md) · [`GameContext.h`](../GameContext.h.md) · [`RoRVersion.h`](../../../source/version_info/RoRVersion.h.md) · [`gameplay/ScriptEvents.h`](../gameplay/ScriptEvents.h.md) · [Seam: HTTP client](../../../SYSTEM-REQUIREMENTS.md#seam-http-client)
**Used by** — callers of [`CurlHelpers.h`](CurlHelpers.h.md) (see its Used by)
**Tier floor** — T2


## Purpose

See [`CurlHelpers.h`](CurlHelpers.h.md).

## State

Stateless.

## `GetUrlAsString`

GET over IPv4 only, accepting gzip, user agent `Rigs of Rods Client/<version>`, OS-native CA store on Windows. Success means transport OK **and** status 200; otherwise the body is replaced with the transport error text and the failure logged.

## `GetUrlAsStringMQ`

Runs the GET and, if the task names a failure message type, posts it with a script-event payload: status CURLSTRING_SUCCESS or CURLSTRING_FAILURE, HTTP status, transport code, body.

**Notes** — the original posts *both* outcomes under the failure message type (the success type is never used), and the download-progress reporter (percent + "Downloading … MB" text on the progress message) exists but is never installed. Receivers therefore distinguish success by the status field; a rebuild should keep that field authoritative whichever message type it uses.
