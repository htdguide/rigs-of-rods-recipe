# source/main/threadpool/ThreadPool.h

> Fixed pool of worker threads running submitted closures, with join handles and a fork-join helper.

**Needs** — [`Application.h`](../Application.h.md) (`app_num_workers` CVar, logging) · [Seam: Thread pool](../../../SYSTEM-REQUIREMENTS.md#seam-thread-pool)
**Used by** — [`Application.cpp`](../Application.cpp.md) · [`gfx/GfxActor.h`](../gfx/GfxActor.h.md) · [`physics/ActorManager.cpp`](../physics/ActorManager.cpp.md) · [`physics/ActorManager.h`](../physics/ActorManager.h.md)
**Tier floor** — T2: real OS threads, mutex and condition variable

## Purpose

The only concurrency primitive RoR builds itself. Two instances exist in practice: the global pool (sized to the machine) used to parallelize per-actor physics and inter-actor collision and to run HTTP requests, and a one-thread pool owned by [`ActorManager`](../physics/ActorManager.cpp.md) that runs the whole physics step concurrently with rendering. Most T2 languages ship an equivalent; the only behaviours that matter are listed below.

## State

```text
RECORD Task
  work      : closure()     # the job
  finished  : bool          # false until work returns
  run_lock  : mutex         # held for the whole duration of work()
  done_cv   : condition

RECORD ThreadPool
  workers   : list<Thread>
  queue     : queue<Task>   # FIFO, shared by all workers
  queue_lock: mutex
  available : condition     # signalled on push and on shutdown
  terminate : bool (atomic)
```

## `DetectNumWorkersAndCreate`

**Contract** — builds the global pool. Worker count comes from the `app_num_workers` setting; if that is below 1 or above the number of logical cores, it is replaced by `clamp(logical_cores − 1, 1, 8)` and the corrected value is written back to the setting so it persists.

**Notes** — one core is left for the main (render) thread; the cap of 8 reflects diminishing returns of the per-actor split, not a hardware limit.

## `RunTask`

**Contract** — enqueues a closure and returns a shared handle immediately. Never blocks except briefly on the queue lock. Tasks start in submission order (single FIFO), but finish in any order.

## `Task.join`

**Contract** — blocks the caller until the task's closure has returned. Safe to call before the task has started, while it runs, or after it finished; spurious wake-ups are absorbed. May be called by several threads.

## `Parallelize`

**Contract** — runs a list of closures concurrently and returns only when all have finished. The **first closure runs on the calling thread**; the rest are submitted to the pool.

```text
FUNCTION parallelize(pool, jobs: list<closure>)
  IF jobs IS EMPTY: RETURN
  handles = [pool.run_task(j) FOR EACH j IN jobs[1..]]
  jobs[0]()                                  # caller does useful work instead of idling
  FOR EACH h IN handles: h.join()
```

**Notes** — running the first job inline is load-bearing: when `Parallelize` is itself called from a pool worker (the physics thread is one), a pool with N workers never deadlocks on N nested fork-joins, because every caller always makes progress on at least one job.

## Worker loop and shutdown

```text
FUNCTION worker(pool)
  LOOP
    LOCK pool.queue_lock DURING
      WHILE pool.queue IS EMPTY
        IF pool.terminate: RETURN
        WAIT pool.available
      task = POP pool.queue
    LOCK task.run_lock DURING
      task.work()
      task.finished = true
    NOTIFY ALL task.done_cv
```

Destroying the pool sets `terminate`, wakes all workers, and joins them. Workers finish whatever is queued before exiting only if they reach it before seeing `terminate` with an empty queue — i.e. shutdown drains the queue.
