# source/main/utils/memory — shared ownership with the script engine

Second chapter. Depends only on the main-thread id from [`AppContext`](../../AppContext.h.md) (a check, not a real dependency).

Every long-lived game object that scripts may hold — actors, cache entries, the terrain, engines, sound scripts — is reference-counted intrusively so that host code and the AngelScript engine share one count. A rebuild in a garbage-collected language still needs this mechanism at the script boundary if it embeds the native script engine; internally it can use its own ownership.

| Twin | Role |
|---|---|
| [`RefCountingObject.h`](RefCountingObject.h.md) | The count and self-destruction, main-thread only |
| [`RefCountingObjectPtr.h`](RefCountingObjectPtr.h.md) | The owning handle, also registered as a script handle type |
