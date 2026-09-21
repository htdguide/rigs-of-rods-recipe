# source/main/system/Console.cpp

> Message history and the two-way bridge between console and log file.

**Needs** — [`Console.h`](Console.h.md) · [`Application.h`](../Application.h.md) · [`utils/Utils.h`](../utils/Utils.h.md)
**Used by** — callers of [`Console.h`](Console.h.md) (see its Used by)
**Tier floor** — T2

## Purpose

Every console message is also logged, and (optionally) every log line is echoed to the console; this file prevents the loop and formats log lines.

## State

See header twin.

## `messageLogged` (log listener)

**Contract** — when `diag_log_console_echo` is on, each log line is forwarded to the console with area LOG: engine warnings → SYSTEM_WARNING, critical → SYSTEM_ERROR, everything else → SYSTEM_NOTICE; text is UTF-8-sanitised.

## `handleMessage(area, type, text, net_user_id, icon)`

```text
FUNCTION handle_message(area, type, text, net_id, icon)
  IF net_id < 0: net_id = 0
  IF area != LOG AND type != SYSTEM_NETCHAT           # no echo loop; chat is never logged (privacy)
    LOG "[RoR|" + {INFO:"General", SCRIPT:"Script", ACTOR:"Actor", TERRN:"Terrn"}[area] + "|"
        + {NOTICE:"Notice", ERROR:"Error", WARNING:"Warning", REPLY:"Success"}[type] + "] " + text
  LOCK messages DURING
    APPEND Message(area, type, text, timer.now_ms, net_id, icon)
```

Unlisted areas/types contribute empty text in the log prefix (e.g. `[RoR||] …` for HELP messages).

## `putMessage` / `putNetMessage`

**Contract** — local messages use user id 0; network messages use area INFO and the sender's id.

## `purgeNetChatMessagesByUser(id)`

**Contract** — removes every message whose user id equals `id` (used when muting a player).
