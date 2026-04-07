---
description: A reactive effect watches activeElement() and immediately disables the queue on any keypress; the queue re-enables only on the renderer's 'idle' event
type: architecture
module: core
created: 2026-04-07
---

# task queue suspends on focus change and resumes when renderer is idle

The task queue in `render.ts` implements idle-only background processing. When a user presses a key, focus changes, and all scheduled background work is paused until rendering settles.

The mechanism uses a `createRenderEffect` that tracks the `activeElement()` signal:

```ts
createRoot(() => {
  createRenderEffect(() => {
    activeElement(); // track the signal
    tasksEnabled = false;
  });
});
```

Any change to the focused element sets `tasksEnabled = false`. This immediately halts further task execution even if the queue has pending items.

Tasks resume when the renderer emits the `'idle'` event, set up in `createRenderer()`:

```ts
renderer.on('idle', () => {
  tasksEnabled = true;
  processTasks();
});
```

The `processTasks()` function uses `setTimeout(task, Config.taskDelay || 50)` — not a microtask — so each task runs one per frame with a configurable delay between tasks. This prevents background work from blocking responsive navigation.

Since [[scheduleTask enables deferred background work with high and low priority]] queuing, callers can push work without worrying about contention with active focus changes.

---

Related Entries:
- [[LazyRow and LazyColumn incrementally render items one per frame until upCount is reached]] — eagerLoad mode uses the task queue to pre-render all items in the background; suspension ensures navigation stays responsive during loading
- [[Lazy delay prop debounces item loading during rapid navigation]] — complements task queue suspension by also pausing loading during rapid directional key presses

Source: [[render]]
Domains:
- [[core guide]]
- [[performance guide]]
