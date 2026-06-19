# Beyond the Basics of Structured Concurrency
**WWDC23 · Session 10170** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10170/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17, watchOS 10, visionOS 1

## Overview
This session moves past the introductory Swift concurrency material to explore advanced patterns enabled by the structured task tree. It covers three interconnected topics: how the task tree powers automatic cancellation and priority propagation, patterns for managing resource usage with task groups, and how task-local values enable observability in distributed systems.

The session introduces two significant new APIs in Swift 5.9: `withDiscardingTaskGroup` / `withThrowingDiscardingTaskGroup` — task groups that immediately free child task resources on completion without needing result collection — and integration with the `swift-distributed-tracing` package for cross-node tracing of server workloads. The `MetadataProvider` API in SwiftLog 1.5 is also demonstrated as a way to automatically inject task-local context into log messages.

## Key Topics

### The Task Tree
- Structured tasks (`async let`, task groups) form a parent-child tree; unstructured tasks (`Task`, `Task.detached`) do not.
- Always prefer structured tasks — cancellation, priority propagation, and task-local value inheritance all flow automatically through the tree.

### Task Cancellation
- **Automatic cancellation**: structured tasks are cancelled when they go out of scope; `group.cancelAll()` cancels all active and future child tasks.
- **Explicit cancellation**: unstructured tasks need `task.cancel()`.
- Cancellation is **cooperative** — it sets `isCancelled` but doesn't stop execution; code must check `Task.isCancelled` or call `Task.checkCancellation()`.
- `withTaskCancellationHandler(operation:onCancel:)` — responds to cancellation while a task is suspended (e.g., inside an `AsyncSequence`'s `next()`). The `onCancel` closure runs synchronously and immediately; shared mutable state must be protected (use atomics, dispatch queues, or locks, not actors, since actor ordering isn't guaranteed).

### Task Priority Propagation
- Child tasks inherit priority from parent by default.
- Awaiting a result from a higher-priority task **escalates** the priority of all child tasks in the tree (priority inversion prevention). Escalation is permanent — it cannot be undone.
- Awaiting `group.next()` escalates all children (since any could complete next).

### Patterns with Task Groups
- **Limiting concurrency**: seed the group with `maxConcurrentTasks` initial tasks; inside the result-collection loop, add a new task each time one completes. This "sliding window" pattern prevents task explosion.
- **`withDiscardingTaskGroup`** / **`withThrowingDiscardingTaskGroup`** **[NEW in Swift 5.9]**: child task resources are freed immediately on completion — no need to collect results. Ideal for fire-and-forget workloads (e.g., processing a stream of server requests). Supports **automatic sibling cancellation**: if any child throws, all remaining children are cancelled automatically.

### Task-Local Values
- Declared with `@TaskLocal` property wrapper on `static` properties; type should be optional to provide a nil default.
- Bound with `TaskLocalKey.$property.withValue(value) { ... }` — binding is scoped; reverts at scope exit.
- **Inheritance**: all structured child tasks automatically inherit task-local values from the parent; `Task.detached` does not inherit.
- **Shadowing**: rebinding within a child scope shadows the parent value; the parent value is restored when the child scope exits.
- Lookup walks the task tree from current task to root; Swift runtime optimizes this with a direct reference.

### Distributed Observability
- **SwiftLog `MetadataProvider`** (new in SwiftLog 1.5): a closure that reads task-local values and returns them as `Logger.Metadata`; automatically injected into every log call without manual boilerplate. Combine multiple providers with `MetadataProvider.multiplex([...])`.
- **Swift Distributed Tracing package** (`swift-distributed-tracing`): implements OpenTelemetry protocol; integrates with Zipkin, Jaeger, and other tracing backends. Uses `withSpan(_:)` to annotate regions of code with named spans. Spans carry timing, relationships, error info, and custom attributes.
- Distributed tracing uses task-local values internally to propagate trace context across async suspension points and even across machines.

## APIs & Frameworks

### Swift Concurrency (Swift 5.9)
- `async let` — structured child task creation
- `withTaskGroup(of:returning:body:)` — throwing/non-throwing task group
- `withThrowingTaskGroup(of:returning:body:)` — throwing task group
- `withDiscardingTaskGroup(body:)` **[NEW]** — no result collection; immediate resource release
- `withThrowingDiscardingTaskGroup(body:)` **[NEW]** — throwing variant; automatic sibling cancellation on error
- `TaskGroup.addTask(priority:operation:)` — adds a child task
- `TaskGroup.next()` — awaits next completed child
- `TaskGroup.cancelAll()` — cancels all children
- `Task.isCancelled` — read-only Bool; check before expensive work
- `Task.checkCancellation()` — throws `CancellationError` if cancelled
- `withTaskCancellationHandler(operation:onCancel:)` — cancellation handler for suspended tasks
- `CancellationError` — standard error type for cancelled tasks
- `@TaskLocal` property wrapper — declares a task-local value
- `TaskLocal.$property.withValue(_:operation:)` — scoped binding
- `Task.sleep(for:)` — suspending sleep

### Swift Atomics Package
- `ManagedAtomic<T>` — atomic value for protecting shared state in cancellation handlers
- `.load(ordering:)` / `.store(_:ordering:)` — atomic read/write

### SwiftLog (v1.5)
- `Logger.MetadataProvider` **[NEW in SwiftLog 1.5]** — closure-based metadata injection
- `Logger.MetadataProvider.multiplex([...])` **[NEW]** — combines multiple providers
- `LoggingSystem.bootstrap(_:metadataProvider:)` — initialize logging system with metadata provider
- `Logger(label:)` — create a logger instance

### Swift Distributed Tracing
- `withSpan(_:body:)` — annotates a scope with a named span; propagates trace context
- `Span.attributes[key]` — attach custom metadata to a span
- OpenTelemetry protocol support; compatible with Zipkin, Jaeger

### Instruments
- Swift Concurrency instrument — visualizes task tree relationships

## Code Highlights

Discarding task group with automatic sibling cancellation:
```swift
func run() async throws {
    try await withThrowingDiscardingTaskGroup { group in
        for cook in staff.keys {
            group.addTask { try await cook.handleShift() }
        }
        group.addTask {
            try await Task.sleep(for: shiftDuration)
            throw TimeToCloseError()  // cancels all other children
        }
    }
}
```

Task-local values for distributed logging:
```swift
actor Kitchen {
    @TaskLocal static var orderID: Int?
    @TaskLocal static var cook: String?
}

// Bind and auto-propagate to child tasks
await Kitchen.$cook.withValue("Sakura") {
    try await makeSoup(order)
}
```

Distributed tracing span with custom attributes:
```swift
try await withSpan(#function) { span in
    span.attributes["kitchen.order.id"] = order.id
    async let pot = stove.boilWater()
    // ...
}
```

## Takeaways
- `withDiscardingTaskGroup` (Swift 5.9) is the right tool for fire-and-forget server workloads — it frees memory immediately and cancels siblings on error automatically.
- Task-local values are the foundation for structured observability: they propagate context automatically through the task tree without passing parameters.
- SwiftLog's `MetadataProvider` + `@TaskLocal` eliminates repetitive manual logging of context (order IDs, trace IDs, etc.).
- `withSpan` from `swift-distributed-tracing` bridges Swift's structured concurrency task tree to distributed OpenTelemetry traces across server nodes.

---
_Source: WWDC23 Session 10170 page (abstract, chapter summaries, code samples, and resource links)._
