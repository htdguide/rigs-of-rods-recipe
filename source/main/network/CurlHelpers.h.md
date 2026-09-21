# source/main/network/CurlHelpers.h

> Blocking HTTP GET into a string, with an optional variant that reports the result to the game as a message.

**Needs** — [`GameContext.h`](../GameContext.h.md)
**Used by** — [`CurlHelpers.cpp`](CurlHelpers.cpp.md) · [`scripting/GameScript.cpp`](../scripting/GameScript.cpp.md)
**Tier floor** — T2


## Purpose

Thin wrapper over the [Seam: HTTP client](../../../SYSTEM-REQUIREMENTS.md#seam-http-client), used by background tasks (repository browser, server list, script HTTP requests). Implementation: [`CurlHelpers.cpp`](CurlHelpers.cpp.md).

## State

```text
RECORD CurlTaskContext
  display name, url
  progress / success / failure message types (INVALID = don't send)
  last reported percentage
```

## `GetUrlAsString(url) → (ok, transport result, http status, body)`

Blocking. See implementation.

## `GetUrlAsStringMQ(task) → ok`

Blocking; posts the outcome to the game's message queue.
