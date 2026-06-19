# Swift Concurrency: Behind the Scenes
**WWDC21 · Session 10254** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10254/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This advanced session dives into the runtime mechanics of Swift's concurrency model, explaining why the language features are designed the way they are — not just for safety, but for performance and efficiency. It builds on "Meet async/await in Swift," "Explore structured concurrency in Swift," and "Protect mutable state with Swift actors."

The session contrasts Swift concurrency with Grand Central Dispatch (GCD), showing how thread explosion arises from GCD's thread-per-blocked-work-item model, and how Swift's cooperative thread pool eliminates this by maintaining a runtime contract that threads never block. It then covers actor internals — how actors compare to serial dispatch queues, actor hopping, reentrancy, and the special behavior of `MainActor`.

The core insight is that Swift's `await` semantics and structured task dependencies together give the runtime enough information to schedule efficiently, keeping thread count bounded to the number of CPU cores.

## Key Topics

### Thread Explosion in GCD
GCD spawns a new thread whenever an existing thread blocks. On a concurrent queue, calling `DispatchQueue.sync` on another serial queue from a completion handler causes each callback to block, spawning more threads. With 100 concurrent tasks, this produces 100+ threads on a 6-core device — 16x overcommitment. Each blocked thread holds memory (stack + kernel data structures) and causes expensive full context switches.

### Swift's Cooperative Thread Pool
Swift concurrency backs tasks with a cooperative thread pool that creates at most as many threads as CPU cores. This is possible because `await` never blocks a thread — instead, it suspends the current task, stores its state as a continuation on the heap, and frees the thread to execute other work. When the awaited work completes, any available thread picks up the continuation.

### Async Function Stack Layout
Async functions use both a traditional OS stack frame (for locals that don't span suspension points) and async frames on the heap (for values that must survive an `await`). This heap-allocated continuation chain is the Swift runtime representation of an async task's pending work. It enables safe, cheap suspension and resumption across arbitrary threads.

### Runtime Contract and Safe Primitives
The cooperative thread pool requires that threads always make forward progress. Swift concurrency primitives (`await`, actors, task groups) maintain this contract at compile time. `os_unfair_lock` and `NSLock` are safe for tight critical sections (the lock-holder always progresses). `DispatchSemaphore` and `NSCondition` are **unsafe** — they block a thread indefinitely without the runtime knowing about the dependency. An environment variable (`LIBDISPATCH_COOPERATIVE_POOL_STRICT=1`) enables a debug runtime that detects violations.

### Actor Internals and Actor Hopping
Actors replace serial dispatch queues. When calling an uncontended actor, the calling thread hops directly to the actor's execution context without a context switch. When an actor is contended, the calling task suspends (thread is freed) and the new work item is queued on the actor. This is fully nonblocking — unlike `DispatchQueue.sync` — and avoids thread explosion.

### Actor Reentrancy and Priority
Actors support reentrancy: while one work item is suspended (awaiting), another work item can begin executing on the same actor. This breaks strict FIFO ordering, enabling the runtime to prioritize higher-priority work items ahead of lower-priority suspended items — directly solving priority inversion that serial dispatch queues suffer from.

### MainActor Performance
`MainActor` abstracts the main thread, which is disjoint from the cooperative pool. Hopping to/from `MainActor` inside a loop causes two full thread context switches per iteration. Batching main-actor work (passing arrays instead of individual values) dramatically reduces context switch overhead.

## APIs & Frameworks

**Swift Concurrency (all new in Swift 5.5 / iOS 15)**
- `async` / `await` **[NEW]** — suspends a task at a potential suspension point without blocking the thread
- `withThrowingTaskGroup(of:body:)` **[NEW]** — creates a structured task group of child tasks
- `TaskGroup.async { }` **[NEW]** — spawns a child task within a group
- `actor` **[NEW]** — reference type that serializes access to its mutable state; replaces serial dispatch queues
- `@MainActor` **[NEW]** — attribute/global actor that isolates code to the main thread
- Continuation (heap-allocated async frame) — runtime representation of suspended async work
- Cooperative thread pool — new default executor for Swift async tasks; capped at CPU core count

**Grand Central Dispatch (contrasted)**
- `DispatchQueue` (serial and concurrent) — thread-per-blocked-item model; subject to thread explosion
- `DispatchQueue.sync` — blocks the calling thread; unsafe to use across actors
- `DispatchSemaphore` — **unsafe** with Swift concurrency (hides dependencies from runtime)
- `NSCondition` — **unsafe** with Swift concurrency

**Safe Synchronization Primitives**
- `os_unfair_lock` — safe for tight critical sections in synchronous code
- `NSLock` — safe for tight critical sections in synchronous code

**URLSession**
- `URLSession.shared.data(from:)` **[NEW async variant]** — async/await interface for network requests

**Instruments**
- System Trace — recommended for profiling Swift concurrency adoption

**Debug Environment Variable**
- `LIBDISPATCH_COOPERATIVE_POOL_STRICT=1` — detects unsafe blocking in the cooperative thread pool

## Code Highlights

GCD approach (thread explosion risk):
```swift
let urlSession = URLSession(configuration: .default, delegate: self, delegateQueue: concurrentQueue)
for feed in feedsToUpdate {
    urlSession.dataTask(with: feed.url) { data, _, _ in
        let articles = try! deserializeArticles(from: data!)
        databaseQueue.sync { updateDatabase(with: articles, for: feed) }  // blocks thread
    }.resume()
}
```

Swift concurrency equivalent (no thread explosion):
```swift
await withThrowingTaskGroup(of: [Article].self) { group in
    for feed in feedsToUpdate {
        group.async {
            let (data, _) = try await URLSession.shared.data(from: feed.url)
            let articles = try deserializeArticles(from: data)
            await updateDatabase(with: articles, for: feed)
            return articles
        }
    }
}
```

Batching MainActor work to reduce context switches:
```swift
// Bad: 2 context switches per loop iteration
@MainActor func updateArticles(for ids: [ID]) async throws {
    for id in ids {
        let article = try await database.loadArticle(with: id)
        await updateUI(for: article)
    }
}
// Good: 2 context switches total
@MainActor func updateArticles(for ids: [ID]) async throws {
    let articles = try await database.loadArticles(with: ids)
    await updateUI(for: articles)
}
```

## Takeaways
- Swift's cooperative thread pool caps thread count at CPU core count; `await` suspends tasks (not threads), enabling this without deadlock or starvation.
- Never use `DispatchSemaphore` or `NSCondition` inside Swift concurrency contexts — they block threads and violate the forward-progress contract; use `LIBDISPATCH_COOPERATIVE_POOL_STRICT=1` to detect violations in testing.
- Actors replace serial queues with nonblocking, reentrancy-aware mutual exclusion that also resolves priority inversion; actor hopping within the cooperative pool is cheap (no context switch in the uncontended case).
- Minimize hops to/from `MainActor` by batching work into arrays; each hop to the main thread costs a full OS thread context switch.

---
_Source: WWDC21 Session 10254 page (abstract, chapter summaries, code samples, and resource links)._
