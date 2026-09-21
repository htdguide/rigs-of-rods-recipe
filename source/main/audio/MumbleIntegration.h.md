# source/main/audio/MumbleIntegration.h

> Publishes player and camera pose to the Mumble voice client via its shared-memory link, so voice chat is positional.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`Application.cpp`](../Application.cpp.md) · [`MumbleIntegration.cpp`](MumbleIntegration.cpp.md) · [`main.cpp`](../main.cpp.md)
**Tier floor** — T2


## Purpose

The [Seam: Mumble positional link](../../../SYSTEM-REQUIREMENTS.md#seam-mumble-positional-link). Compiled only with Mumble support. Implementation: [`MumbleIntegration.cpp`](MumbleIntegration.cpp.md).

## State

```text
RECORD LinkedMem            # layout owned by Mumble ("Link" plugin v2); must match byte for byte
  uiVersion u32, uiTick u32
  avatar position[3], front[3], top[3] : f32
  name        : wide char[256]
  camera position[3], front[3], top[3] : f32
  identity    : wide char[256]
  context_len : u32, context : byte[256]
  description : wide char[2048]
RECORD MumbleIntegration
  link : pointer to the mapped LinkedMem, or none (Mumble not running at startup)
```

**Notes** — wide char is 2 bytes on Windows and 4 on Linux/macOS; the layout follows the platform's native wide char because Mumble reads it the same way.

## `MumbleIntegration()`

Maps the link on construction; failure leaves `link` empty and every later call a no-op. There is no retry, so Mumble must be started before the game.

## `Update()`

Once per frame. See the implementation.
