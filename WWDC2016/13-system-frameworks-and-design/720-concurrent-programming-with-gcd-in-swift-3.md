# Concurrent Programming With GCD in Swift 3
**WWDC16 · Session 720** · [Watch](https://developer.apple.com/videos/play/wwdc2016/720/)

_Platforms:_ iOS 10, macOS Sierra 10.12, watchOS 3, tvOS 10

## Overview
This session teaches how to structure Swift 3 applications using Grand Central Dispatch to achieve safe, efficient concurrency. It covers the fundamentals of dispatch queues and work submission, recommended application architecture patterns (subsystems backed by queues), Quality of Service classes, and practical synchronization techniques. The second half focuses on object lifecycle management in concurrent environments—setup, activation, invalidation, and deallocation—using a four-step state machine to avoid the deadlocks and abandoned memory that are common in heavily concurrent code.

GCD's Swift 3 API is substantially redesigned with new value-type wrappers, new activation/precondition APIs, and a new `os_unfair_lock` replacing the problematic `OSSpinLock`. The session emphasizes that there is no such thing as a benign data race; every shared mutable state must be properly synchronized.

## Key Topics

### Application Structure with Dispatch
- Identify independent data flows in your app and back each subsystem with its own serial `DispatchQueue`.
- Use async submission to move expensive work (image processing, data transforms) off the main thread; chain back to `DispatchQueue.main` for UI updates.
- Thread explosion: blocking worker threads causes dispatch to spawn more threads; keep queue count manageable.

### Async, Sync, and Groups
- `DispatchQueue.async` — fire-and-forget; dispatch manages the worker thread lifecycle.
- `DispatchQueue.sync` — blocks the calling thread until the work item completes; provides mutual exclusion on a serial queue. Can return a value in Swift 3.
- `DispatchGroup` — submit work items across queues with a shared group; use `group.notify(queue:)` to receive a callback when all items finish.

### Quality of Service
- Pass `DispatchQoS` to `async` or queue init to classify work intent (`.userInteractive`, `.userInitiated`, `.utility`, `.background`).
- When high-QoS work is submitted behind lower-QoS work, dispatch automatically elevates the lower-priority items ahead to resolve priority inversions.
- Create dedicated background queues so all work on that queue runs at the correct priority by default.

### DispatchWorkItem
- `DispatchWorkItem(flags:block:)` — captures execution context (QoS, logging) at creation time rather than submission time (use `.assignCurrentContext` flag).
- `DispatchWorkItem.wait()` — blocks calling thread and causes dispatch to elevate work ahead of the item (unlike semaphores, which have no ownership information).

### Synchronization with Queues
- Prefer `DispatchQueue.sync { }` over locks; code runs in a scoped block, so it is impossible to forget to unlock.
- `dispatchPrecondition(condition: .onQueue(q))` **[NEW]** — asserts that code runs on a specific queue; triggers a crash with useful information if violated.
- `dispatchPrecondition(condition: .notOnQueue(q))` **[NEW]** — asserts that code does NOT run on a specific queue (prevents deadlock-prone patterns).
- `os_unfair_lock` **[NEW]** — non-spinning replacement for the deprecated `OSSpinLock`; not prone to priority inversions.
- In Swift 3, global variables are initialized atomically, but class properties and lazy class properties are NOT atomic and require explicit synchronization.

### Object Lifecycle (Setup → Activate → Invalidate → Deallocate)
- **Setup**: initialize object, set properties (handlers, labels, attributes).
- **Activate**: make the object known to other subsystems; start receiving events/notifications. Use `DispatchSource.activate()` **[NEW]** rather than the first `resume()`.
- **Invalidate**: explicit invalidation function (not `deinit`) must deregister from all other subsystems; track invalidation with a stored Boolean; enforce with `dispatchPrecondition`.
- **Deallocate**: `deinit` only runs cleanup, not deregistration. Deregistering from `deinit` can cause deadlocks (if the dealloc runs on a queue that the object is synchronized with) or abandoned memory (if other subsystems still hold references).
- Dispatch objects follow the same lifecycle; dispatch asserts in debug builds if an object is deallocated while inactive or suspended.

### DispatchSource Lifecycle
- `DispatchSource.makeReadSource(fileDescriptor:queue:)` etc. — create initially inactive with `.initiallyInactive` attribute **[NEW]**.
- `DispatchSource.activate()` **[NEW]** — replaces initial `resume()`; separates activation from suspension/resume semantics; idempotent.
- `DispatchSource.cancel()` — stops event delivery, runs cancellation handler on the target queue (use handler to close file descriptors, free resources); destroys all handlers (breaking retain cycles).
- Always cancel sources before releasing them; always activate queues/sources before deallocation.

## APIs & Frameworks

- **Dispatch (GCD)** — Swift 3 value-type redesign **[NEW]**
- `DispatchQueue(label:attributes:qos:target:)` — create a serial or concurrent queue
- `DispatchQueue.async(group:qos:flags:execute:)` — asynchronous submission
- `DispatchQueue.sync(execute:)` / `DispatchQueue.sync(flags:execute:) -> T` — synchronous submission (can return a value) **[NEW in Swift 3]**
- `DispatchQueue.main` — main thread queue
- `DispatchQueue.global(qos:)` — system global concurrent queues
- `DispatchGroup()` — group for tracking sets of work items
- `DispatchGroup.async(group:queue:execute:)` — submit to group
- `DispatchGroup.notify(queue:execute:)` — callback when group empties
- `DispatchGroup.wait()` — synchronously wait for group to empty
- `DispatchWorkItem(qos:flags:block:)` — explicit work item with context capture **[NEW in Swift 3]**
- `DispatchWorkItem.wait()` — wait with priority-inheritance **[NEW in Swift 3]**
- `DispatchQoS` — `.userInteractive`, `.userInitiated`, `.default`, `.utility`, `.background`, `.unspecified`
- `DispatchWorkItemFlags.assignCurrentContext` — capture QoS at creation time
- `dispatchPrecondition(condition:)` **[NEW]** — `.onQueue`, `.notOnQueue`
- `DispatchSource.activate()` **[NEW]** — explicit activation separate from resume
- `DispatchSource.cancel()` — cancel and destroy handlers
- `DispatchSource.setCancelHandler(handler:)` — resource cleanup handler
- `DispatchQueue.Attributes.initiallyInactive` **[NEW]** — create queue inactive for staged configuration
- `os_unfair_lock` / `os_unfair_lock_lock` / `os_unfair_lock_unlock` **[NEW]** — non-spinning, priority-inversion-safe lock
- `Foundation.NSLock` — alternative lock class safe to use from Swift
- `OSSpinLock` — **deprecated**

## Code Highlights

Moving work off the main thread and returning to main:
```swift
let transformQueue = DispatchQueue(label: "com.myapp.transform")

transformQueue.async {
    let result = self.transform(data)
    DispatchQueue.main.async {
        self.imageView.image = result
    }
}
```

Returning a value from a sync call (new in Swift 3):
```swift
let value = queue.sync { return self.internalState }
```

Using dispatch preconditions to enforce queue invariants:
```swift
func updateUI() {
    dispatchPrecondition(condition: .onQueue(DispatchQueue.main))
    // safe to update UI
}
```

Explicit invalidation to avoid deadlock in `deinit`:
```swift
func invalidate() {
    dispatchPrecondition(condition: .onQueue(DispatchQueue.main))
    isInvalidated = true
    subsystem.unregister(self)
}
```

## Takeaways
- Back each independent subsystem with its own serial `DispatchQueue`; chain async calls between queues rather than creating many queues and blocking threads, which causes thread explosion.
- Use `DispatchWorkItem.wait()` instead of semaphore waits when you need priority inheritance; semaphores have no ownership information and do not propagate QoS.
- Implement an explicit `invalidate()` method for objects used concurrently; never deregister from other subsystems in `deinit` — this is a source of both deadlocks and abandoned memory and now asserts in debug builds.
- Always cancel `DispatchSource` objects before releasing them to destroy their handlers and break retain cycles; always activate dispatch objects before deallocating.

---
_Source: WWDC16 Session 720 page (abstract, transcript, and resource links)._
