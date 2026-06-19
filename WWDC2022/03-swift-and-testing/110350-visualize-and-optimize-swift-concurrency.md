# Visualize and Optimize Swift Concurrency
**WWDC22 · Session 110350** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110350/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This session introduces the new Swift Concurrency template in Instruments 14, which provides first-class visualization of tasks, actors, and continuations. Using a "File Squeezer" compression app as a running example, it identifies and fixes two real performance problems — a `@MainActor` UI hang and actor contention preventing parallel execution — and explains two additional pitfalls: thread pool exhaustion and continuation misuse.

## Key Topics

### Swift Concurrency Recap
Swift Concurrency consists of four interlocking primitives:
- **Async/await** — functions that suspend without blocking a thread
- **Tasks** — the unit of work; own local state, handle cancellation
- **Structured concurrency** — child tasks (task groups, `async let`) that run in parallel and are awaited by their parent
- **Actors** — isolate shared mutable state so only one task accesses it at a time

### Swift Concurrency Instruments (New in Instruments 14)
The new **Swift Concurrency** template includes two instruments:

**Swift Tasks instrument**
- *Running Tasks* track — tasks executing simultaneously (parallelism indicator)
- *Alive Tasks* track — tasks in memory at any point in time (memory pressure indicator)
- *Total Tasks* counter — cumulative task creation count
- *Task Forest* detail view — parent/child graph for structured concurrency
- *Task Summary* detail view — time each task spends in each state (Running, Enqueued, Suspended, etc.)
- Pin-to-timeline action — right-click any task to pin a dedicated timeline track; reveals narrative view (explains what the task is waiting for and which other task/actor it is blocked on) and creation backtrace in the Extended Detail view

**Swift Actors instrument**
- Per-actor track showing queue depth and time tasks spend waiting to enter
- Annotates time intervals spent executing on each actor

### Problem 1: Main Actor Blocking
If a long-running synchronous computation runs on a `@MainActor`-isolated class or function, the main thread is blocked, causing a visible UI hang. Symptoms in Instruments: Running Tasks count is consistently 1; a long-running task is shown executing on the main thread.

Fix: move CPU-intensive work into its own `actor`, keeping only the `@Published` property on `@MainActor`. Use `Task { @MainActor in ... }` to hop back to the main actor only for UI updates.

### Problem 2: Actor Contention
When many tasks all call the same actor's methods, the actor serializes them. If the actor method itself is long-running (e.g., it performs the actual compression), the *Enqueued* state in the Task Summary dominates and the thread pool is underutilized.

Fix: mark the expensive method `nonisolated`, making it execute on the thread pool. Access actor-isolated state only at the brief points where it is truly needed (log appends, UI updates). Use `Task.detached` for task creation to avoid inheriting the calling context's actor isolation, allowing the runtime to schedule each task on any available thread.

### Problem 3: Thread Pool Exhaustion
Swift concurrency's cooperative thread pool is sized to match the number of CPU cores. Tasks that block a thread without suspending (blocking I/O, semaphores, condition variables, long-held locks) occupy a thread without doing CPU work. If all threads are occupied by blocked tasks, the runtime cannot make forward progress and may deadlock.

Rules:
- Use `async` APIs for all file and network I/O
- Avoid `DispatchSemaphore`, condition variables, or blocking locks inside tasks
- If blocking code is unavoidable, run it on a `DispatchQueue` and bridge using continuations

### Problem 4: Continuation Misuse
`withCheckedContinuation` / `withUnsafeContinuation` bridge callback-based code to Swift concurrency. The continuation **must** be resumed exactly once. Calling it twice crashes; never calling it leaks the task indefinitely.

Always prefer `withCheckedContinuation` — it auto-detects misuse: double-resume traps; missed resume prints a warning when the continuation is deallocated. The Instruments task track will show the task stuck in the "Continuation" state if the resume is never called.

## APIs & Frameworks

**Instruments 14 (tooling)**
- Swift Concurrency template **[NEW]**
- Swift Tasks instrument **[NEW]** — Running/Alive/Total Tasks tracks, Task Forest, Task Summary, pin-to-timeline, narrative view
- Swift Actors instrument **[NEW]** — per-actor queue depth and execution tracks

**Swift Concurrency**
- `@MainActor` — isolates class/function to the main thread
- `actor` — custom actor type for thread-safe shared state
- `nonisolated` — removes actor isolation from a method so it runs on the thread pool
- `Task { }` — unstructured task; inherits actor context from creation site
- `Task.detached { }` — task that does not inherit actor context
- `Task { @MainActor in ... }` — hops onto the main actor for a unit of work
- `withCheckedContinuation(_:)` — bridging from callback to `async`, with misuse detection
- `withUnsafeContinuation(_:)` — same, without overhead; use only when performance is critical

## Code Highlights

Before (problematic): entire `CompressionState` class on `@MainActor`, causing the heavy `compressFile` call to block the main thread:
```swift
@MainActor
class CompressionState: ObservableObject {
    @Published var files: [FileStatus] = []
    var logs: [String] = []

    func compressAllFiles() {
        for file in files {
            Task {
                let compressedData = compressFile(url: file.url)  // runs on main thread!
                await save(compressedData, to: file.url)
            }
        }
    }
}
```

After (fixed): `compressFile` marked `nonisolated`, tasks created as detached:
```swift
actor ParallelCompressor {
    var logs: [String] = []
    unowned let status: CompressionState

    nonisolated func compressFile(url: URL) async -> Data {
        await log(update: "Starting for \(url)")
        let compressedData = CompressionUtils.compressDataInFile(at: url) { uncompressedSize in
            Task { @MainActor in status.update(url: url, uncompressedSize: uncompressedSize) }
        } progressNotification: { progress in
            Task { @MainActor in status.update(url: url, progress: progress) }
        } finalNotificaton: { compressedSize in
            Task { @MainActor in status.update(url: url, compressedSize: compressedSize) }
        }
        await log(update: "Ending for \(url)")
        return compressedData
    }

    func log(update: String) { logs.append(update) }
}

@MainActor
class CompressionState: ObservableObject {
    func compressAllFiles() {
        for file in files {
            Task.detached {  // detached = no inherited actor context
                let compressedData = await self.compressor.compressFile(url: file.url)
                await save(compressedData, to: file.url)
            }
        }
    }
}
```

Safe continuation usage:
```swift
let result = await withCheckedContinuation { continuation in
    someCallbackAPI { value in
        continuation.resume(returning: value)  // called exactly once
    }
}
```

## Takeaways
- The new Swift Concurrency template in Instruments 14 provides task-level visibility: Running Tasks count reveals parallelism; the Task Summary Enqueued state reveals actor contention; narrative view explains exactly which actor or task is blocking progress.
- `@MainActor` classes should contain only state that genuinely requires the main thread (`@Published` SwiftUI bindings); move all CPU-intensive work into a separate `actor` or `nonisolated` function.
- Mark actor methods `nonisolated` and use `Task.detached` to allow compression/processing work to run freely on the thread pool, touching the actor only for brief guarded updates.
- Always use `withCheckedContinuation` when bridging callbacks; resuming a continuation zero or more than once is a hard bug — checked continuations detect and report both cases.

---
_Source: WWDC22 Session 110350 page (abstract, transcript, and code samples)._
