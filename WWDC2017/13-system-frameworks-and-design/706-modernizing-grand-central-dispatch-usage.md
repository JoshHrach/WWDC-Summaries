# Modernizing Grand Central Dispatch Usage
**WWDC17 · Session 706** · [Watch](https://developer.apple.com/videos/play/wwdc2017/706/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, watchOS 4, tvOS 11

## Overview
macOS High Sierra and iOS 11 fundamentally reinvented how Grand Central Dispatch collaborates with the Darwin kernel through a new internal mechanism called Unified Queue Identity. This advanced session explains how modern CPUs suffer when code goes off-core unnecessarily (losing branch-prediction and cache history), then lays out the patterns that cause excessive context switching and how to eliminate them. The improvements Apple found from applying these techniques in their own frameworks resulted in 1.3x performance gains from simple structural changes.

The session covers three root causes of excessive context switching: contended lock acquisition (solved with `os_unfair_lock` over POSIX mutexes), switching between independent operations (solved with serial queue hierarchies with a shared mutual exclusion context at the bottom), and the thread explosion pattern from blocking on global concurrent queues (solved with fixed-count subsystem hierarchies). Unified Queue Identity eliminates previously unavoidable thread bounces when related work items target the same serial queue hierarchy, reducing two context switches to zero in many common patterns.

Two concrete modernization rules emerge: never mutate dispatch objects after calling `activate()`, and always use `dispatch_queue_create_with_target` to lock down target queue relationships atomically. A new GCD performance instrument (coming in a later Xcode 9 seed) can detect violations automatically.

## Key Topics
- **Parallelism vs. concurrency** — parallelism = simultaneous multi-core execution; concurrency = interleaved execution of independent subsystems (possible on one core)
- **`DispatchQueue.concurrentPerform` / `dispatch_apply(DISPATCH_APPLY_AUTO)`** — parallel for-loop with automatic load balancing; use large iteration counts (1000+) for flexibility; `DISPATCH_APPLY_AUTO` **[NEW]** lets system choose execution context automatically
- **Context switch cost** — each switch loses CPU core history (caches, branch predictor); repeated switching between short work items is more harmful than beneficial
- **Three causes of excessive context switching**: (1) lock contention on fair locks; (2) many independent queues becoming active simultaneously; (3) thread explosion from blocking on global concurrent queue
- **`os_unfair_lock`** — unfair lock; when blue thread re-acquires the lock it stays on-CPU without a context switch; best for object properties and frequently-toggled global state
- **Lock ownership** — serial queues and `os_unfair_lock` have a single known owner; the runtime uses this to resolve priority inversions and perform directed CPU handoffs; `DispatchSemaphore` and `DispatchGroup` have no single owner and cannot benefit
- **Serial queue hierarchy** — form trees: sources and per-object queues target a shared serial exclusion queue at the bottom; results in one worker thread for the whole subsystem instead of one per object
- **One queue hierarchy per subsystem** — fixed count of hierarchies; not one queue per object; may have a primary + secondary hierarchy within a subsystem for responsiveness vs. throughput
- **Work item sizing** — use large items when moving between subsystems (allows CPU to reach efficient state); fine-grained subdivision is acceptable within a single subsystem hierarchy
- **Unified Queue Identity (new in macOS 10.13 / iOS 11)** — single kernel object represents both async and sync work on a queue; enables early priority inversion resolution; enables directed handoff for `dispatch_sync`; eliminates extra threads for related events targeting the same queue hierarchy
- **No mutation after `activate()`** — snapshot of target queue hierarchy is taken at activate time; mutating `targetQueue` or handlers after activation defeats priority inversion avoidance, directed handoff, and Unified Queue Identity optimizations
- **`dispatch_queue_create_with_target`** **[NEW API in 2016, now recommended]** — atomic single-step creation + target setting + hierarchy lockdown; prevents post-activation retargeting; Swift 3 users are already in this mode
- **GCD Performance Instrument** **[NEW, upcoming Xcode 9 seed]** — detects "mutation after activation" and "retarget after activation" events in Instruments System Trace
- **`kdebug_signpost_start` / `kdebug_signpost_end`** — annotate custom code regions to appear in Instruments System Trace points track

## APIs & Frameworks

### Dispatch (GCD)
- **`DispatchQueue(label:attributes:target:)`** — create serial or concurrent queue; pass `target:` to set hierarchy at creation time
- **`dispatch_queue_create_with_target(_:_:_:)`** **[Recommended]** — atomically create + set target + lock hierarchy; use instead of create-then-set-target pattern
- **`DISPATCH_APPLY_AUTO`** **[NEW]** — `dispatch_apply` keyword that auto-selects the execution context; replaces manual queue argument
- **`DispatchQueue.concurrentPerform(iterations:execute:)`** — parallel for-loop (Swift); use 1000+ iterations for good load-balancing flexibility
- **`DispatchSource.makeReadSource(fileDescriptor:queue:)`** — event-monitoring primitive; `queue` is the target queue for event handler execution
- **`DispatchSource.activate()`** — snapshot of properties taken at this point; no mutation after this call
- **`DispatchSource.cancel()` + cancel handler** — invalidation pattern; cancel handler is called after the source is fully cancelled
- **`DispatchQueue.sync(_:)` / `dispatch_sync`** — synchronous execution; with Unified Queue Identity, directed CPU handoff eliminates extra context switches
- **`DispatchQueue.async(_:)` / `dispatch_async`** — asynchronous enqueue; QOS from calling thread unless overridden
- **Quality of Service classes** — `.userInteractive` (UI), `.userInitiated` (IN), `.utility` (UT), `.background` (BG); set on queues or work items

### os / Darwin
- **`os_unfair_lock`** — unfair lock; no fairness guarantee but avoids context switches when same thread re-acquires; preferred over `pthread_mutex` for high-frequency lock/unlock of per-object state
- **`kdebug_signpost_start(_:_:_:_:_:)` / `kdebug_signpost_end(_:_:_:_:_:)`** — instrument custom code regions for System Trace

### Instruments
- **System Trace** — shows CPU, thread, and context-switch tracks; essential for diagnosing lock contention and excessive context switching
- **GCD Performance Instrument** **[NEW, upcoming Xcode 9 seed]** — automatically flags `activate()`-after-mutation and post-activation retargeting

## Code Highlights

**Anti-pattern: independent per-connection queues (causes thread explosion)**
```c
// Old pattern — each connection gets its own independent queue
dispatch_queue_t q = dispatch_queue_create("connection_q", DISPATCH_QUEUE_SERIAL);
dispatch_source_t src = dispatch_source_create(DISPATCH_SOURCE_TYPE_READ, fd, 0, q);
// Many connections = many independent threads
```

**Fixed: shared mutual exclusion context via queue hierarchy**
```c
// One exclusion queue for all connections
dispatch_queue_t eq = dispatch_queue_create("network_eq", DISPATCH_QUEUE_SERIAL);

// Each connection targets the shared exclusion queue
dispatch_queue_t q = dispatch_queue_create_with_target(
    "connection_q", DISPATCH_QUEUE_SERIAL, eq);
dispatch_source_t src = dispatch_source_create(DISPATCH_SOURCE_TYPE_READ, fd, 0, q);
dispatch_source_set_event_handler(src, ^{ /* read data */ });
dispatch_activate(src);
// All connections now execute serially on one thread — no thread explosion
```

**`DISPATCH_APPLY_AUTO` for parallel work**
```c
dispatch_apply(1000, DISPATCH_APPLY_AUTO, ^(size_t i) {
    process_chunk(data, i);
});
```

## Takeaways
- Unified Queue Identity in iOS 11 / macOS High Sierra eliminates spurious context switches for related work targeting the same queue hierarchy — but only if you never mutate dispatch objects after `activate()`.
- Use `dispatch_queue_create_with_target` (or Swift `DispatchQueue(label:target:)`) to atomically create and lock down queue hierarchies; never call `dispatch_set_target_queue` after activation.
- Structure concurrency as a fixed number of serial queue hierarchies (one per subsystem, not one per object); this is the single most impactful change for reducing context-switch overhead.
- `os_unfair_lock` is the right choice for frequently-toggled per-object state; POSIX mutexes (fair locks) cause unnecessary context switches when the same thread could immediately re-acquire the lock.

---
_Source: WWDC17 Session 706 page (abstract, chapter summaries, code samples, and resource links)._
