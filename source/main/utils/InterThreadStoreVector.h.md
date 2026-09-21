# source/main/utils/InterThreadStoreVector.h

> Mutex-protected list that one thread appends to and another drains atomically.

**Needs** — [`Application.h`](../Application.h.md)
**Used by** — [`scripting/ScriptEngine.h`](../scripting/ScriptEngine.h.md)
**Tier floor** — T2

## Purpose

The hand-off buffer between the network receive thread and the main thread (incoming packets), and similar producer/consumer pairs. It exists because the main loop wants all pending items at once, per frame, without holding a lock while processing them.

## State

```text
RECORD InterThreadStoreVector<T>
  items : list<T>
  lock  : mutex
```

## `push`

**Contract** — append one item under the lock. Callable from any thread.

## `pull`

**Contract** — under the lock, replace the caller's list with all stored items and empty the store. Order of pushes is preserved.

**Notes** — any concurrent queue with a "drain all" operation is a drop-in substitute.
