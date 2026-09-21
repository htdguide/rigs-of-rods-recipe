# source/main/audio/Sound.h

> A logical sound: one buffer with gain/pitch/position/loop state, bound on demand to a hardware voice.

**Needs** — [`Application.h`](../Application.h.md) · [`utils/memory/RefCountingObject.h`](../utils/memory/RefCountingObject.h.md)
**Used by** — [`Sound.cpp`](Sound.cpp.md) · [`SoundManager.cpp`](SoundManager.cpp.md) · [`SoundManager.h`](SoundManager.h.md) · [`SoundScriptManager.cpp`](SoundScriptManager.cpp.md) · [`SoundScriptManager.h`](SoundScriptManager.h.md) · [`scripting/bindings/SoundScriptAngelscript.cpp`](../scripting/bindings/SoundScriptAngelscript.cpp.md)
**Tier floor** — T2


## Purpose

There are more logical sounds than audio voices. `Sound` holds the desired state; the [`SoundManager`](SoundManager.cpp.md) decides which sounds currently own a hardware source. Implementation: [`Sound.cpp`](Sound.cpp.md).

## State

```text
RECORD Sound = { buffer; gain = 0; pitch = 1; position; velocity; loop; enabled = true; should_play;
                 audibility; hardware index (−1 = no voice); source index (slot in the manager) }
```

## API

`setPitch`, `setGain`, `setPosition`, `setVelocity`, `setLoop`, `setEnabled`, `play`, `stop`, `isPlaying`, getters.
