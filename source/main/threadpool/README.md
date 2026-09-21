# source/main/threadpool — worker pool

Third chapter. Depends only on the hub's setting for worker count.

A single header implementing a fixed-size FIFO worker pool with per-task join handles and a fork-join `Parallelize` that runs the first job on the caller. Used by physics (per-actor substeps, inter-actor collision, the async physics thread) and by networking/HTTP. Replace with the platform's pool if it offers join handles; keep the "first job inline" rule.

| Twin | Role |
|---|---|
| [`ThreadPool.h`](ThreadPool.h.md) | Pool, task handle, fork-join |
