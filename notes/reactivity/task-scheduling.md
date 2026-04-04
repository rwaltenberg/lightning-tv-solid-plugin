# Task Scheduling

> scheduleTask(callback, priority) queues background work. The queue pauses on activeElement change and resumes on renderer 'idle'. Config.taskDelay default 50ms.

**Source**: `solidjs-integration.md` | **Severity**: informational

## Detail

`src/render.ts` implements a priority task queue that prevents UI jank during input handling by deferring background work.

### Behavior

- Tasks are processed **one at a time** with a configurable delay (`Config.taskDelay`, default: 50ms).
- The queue is **paused** whenever the active (focused) element changes (i.e., during key press handling).
- The queue is **resumed** when the Lightning Renderer fires its `'idle'` event.
- Priority `'high'` tasks execute before `'low'` tasks.

### API

```ts
export function scheduleTask(callback: () => void, priority?: 'high' | 'low'): void;
export function setTasksEnabled(enabled: boolean): void;
export function clearTasks(): void;
```

### Config

```ts
Config.taskDelay?: number;  // default: 50 (ms between tasks)
```

### Connection to Active Element

The task queue is automatically disabled when `setActiveElement` is called (focus changes) and re-enabled on renderer idle. This happens internally -- developers do not need to manually pause/resume for input handling.

## Code Example

```ts
import { scheduleTask } from '@lightningtv/solid';

// Schedule low-priority background work
scheduleTask(() => {
  // Prefetch data, preload images, etc.
  prefetchNextPage();
}, 'low');

// Schedule high-priority work (runs before low-priority tasks)
scheduleTask(() => {
  updateCriticalUI();
}, 'high');
```

## Gotchas

- Tasks do NOT execute immediately -- they are deferred and processed with `Config.taskDelay` between them.
- Do not assume tasks execute in the same frame or synchronously.
- Tasks are paused during input handling -- do not use `scheduleTask` for work that must complete before the next render.
- `clearTasks()` removes all pending tasks from the queue.

## Related Notes

- [core/rendering-pipeline.md] -- where the task queue fits in the bootstrap sequence (idle listener)
- [api/config.md] -- Config.taskDelay field
