---
description: scheduleTask(callback, priority) adds callbacks to a setTimeout-based queue; 'high' pushes to front, 'low' to back; only runs when renderer is idle
type: api
module: core
created: 2026-04-07
---

# scheduleTask enables deferred background work with high and low priority

`scheduleTask` is the public API for deferring non-critical work to idle periods between navigation events.

```ts
export function scheduleTask(
  callback: Task,
  priority: 'high' | 'low' = 'low',
): void {
  if (priority === 'high') {
    taskQueue.unshift(callback);
  } else {
    taskQueue.push(callback);
  }
  processTasks();
}
```

High priority tasks jump to the front of the queue. Low priority (default) appends to the back. After adding the task, `processTasks()` is called immediately — but if `tasksEnabled` is false (e.g., user is navigating), the call is a no-op.

Processing uses `setTimeout(task, Config.taskDelay || 50)` — not microtasks. This means:
- One task runs per setTimeout callback
- `Config.taskDelay` controls the inter-task delay (default 50ms)
- Tasks are processed recursively: each processed task calls `processTasks()` again for the next

Supporting functions:
- `clearTasks()` — empties the queue immediately
- `setTasksEnabled(enabled)` — manually override the enabled state

Since [[task queue suspends on focus change and resumes when renderer is idle]], this is safe to use for background loading, prefetching, or lazy initialization.

Since [[LazyRow and LazyColumn incrementally render items one per frame until upCount is reached]], the `eagerLoad: true` mode uses `scheduleTask` to continue loading items past `upCount` without blocking navigation.

---

Source: [[render]]
Domains:
- [[core guide]]
- [[performance guide]]
