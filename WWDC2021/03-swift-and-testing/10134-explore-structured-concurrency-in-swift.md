# Explore Structured Concurrency in Swift
**WWDC21 · Session 10134** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10134/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
Swift 5.5 introduces structured concurrency, extending the principles of structured programming — static scoping, top-to-bottom control flow, and predictable variable lifetimes — to asynchronous and concurrent code. The session builds on the async/await foundation to explain how tasks are organized into hierarchies (task trees), how cancellation propagates automatically, and how to choose the right task type for each situation.

The session distinguishes four task flavors: `async let` (static concurrency), task groups (dynamic concurrency), unstructured tasks (manually managed, inherit context), and detached tasks (manually managed, no inherited context). Each offers a different balance of structure, safety, and flexibility.

## Key Topics

**async let (Concurrent Bindings)**
Adding `async` before a `let` binding creates a child task immediately. The parent task continues executing; the placeholder value is only awaited when first read. If an earlier awaited task throws before the `async let` value is read, Swift automatically cancels and awaits the child task before propagating the error.

**Task Groups**
`withThrowingTaskGroup(of:)` and `withTaskGroup(of:)` allow spawning a dynamic number of child tasks. Tasks begin executing immediately in any order. The parent iterates results with `for try await … in group`. Cancellation within a group propagates automatically. The `group.cancelAll()` method allows explicit group-wide cancellation.

**Cooperative Cancellation**
Cancellation in Swift is cooperative. Marking a task canceled does not halt it; the task must check `Task.checkCancellation()` (throws) or `Task.isCancelled` (Bool) and wind down cleanly. Cancellation propagates to all descendant tasks in the tree.

**Unstructured Tasks**
`Task { }` creates an unscoped task that inherits the actor and priority of the originating context but whose lifetime is not bound to any scope. The task handle (`Task<Success, Failure>`) must be stored and manually canceled when no longer needed.

**Detached Tasks**
`Task.detached(priority:)` creates a task that inherits nothing from its originating context. Useful for background work that should not be tied to any actor and should not be automatically canceled when the launching scope exits.

**`@Sendable` Closures**
Task closures are `@Sendable`, preventing capture of mutable local variables and enforcing that shared values are safe to pass across concurrency boundaries (value types or actors).

## APIs & Frameworks

- **Swift Concurrency** **[NEW]**
- `async let` binding syntax **[NEW]** — spawns a child task; value is `try await`-ed at use site
- `withTaskGroup(of:returning:body:)` **[NEW]** — creates a structured task group
- `withThrowingTaskGroup(of:returning:body:)` **[NEW]** — structured task group that can throw
- `TaskGroup` **[NEW]**
  - `mutating func addTask(priority:operation:)` (also `group.async { }` in session code)
  - `mutating func cancelAll()`
  - `for await` (conforms to `AsyncSequence`)
- `ThrowingTaskGroup` **[NEW]**
- `Task` **[NEW]** (struct)
  - `init(priority:operation:)` — unstructured task, inherits actor/priority
  - `static func detached(priority:operation:) -> Task` **[NEW]**
  - `static func checkCancellation() throws` **[NEW]**
  - `static var isCancelled: Bool` **[NEW]**
  - `func cancel()`
  - `var value: Success` (async property to await result)
- `@Sendable` closure attribute **[NEW]** — enforces sendability of captured values
- `AsyncSequence` protocol — used by `TaskGroup` and `for await` loops
- `URLSession.shared.data(for:)` async overload **[NEW]**
- `UIImage.byPreparingThumbnail(ofSize:)` async method **[NEW]**
- SE-0304: Structured Concurrency
- SE-0317: async let

## Code Highlights

Parallel downloads with `async let`:
```swift
func fetchOneThumbnail(withID id: String) async throws -> UIImage {
    let imageReq = imageRequest(for: id), metadataReq = metadataRequest(for: id)
    async let (data, _) = URLSession.shared.data(for: imageReq)
    async let (metadata, _) = URLSession.shared.data(for: metadataReq)
    guard let size = parseSize(from: try await metadata),
          let image = try await UIImage(data: data)?.byPreparingThumbnail(ofSize: size)
    else { throw ThumbnailFailedError() }
    return image
}
```

Dynamic concurrency with a task group (results collected by parent):
```swift
try await withThrowingTaskGroup(of: (String, UIImage).self) { group in
    for id in ids {
        group.async { return (id, try await fetchOneThumbnail(withID: id)) }
    }
    for try await (id, thumbnail) in group {
        thumbnails[id] = thumbnail
    }
}
```

Cooperative cancellation check:
```swift
try Task.checkCancellation()
// or
if Task.isCancelled { break }
```

Unstructured task (inherits main actor):
```swift
Task {
    let thumbnails = await fetchThumbnails(for: ids)
    display(thumbnails, in: cell)
}
```

Detached background task with nested group:
```swift
Task.detached(priority: .background) {
    withTaskGroup(of: Void.self) { g in
        g.async { writeToLocalCache(thumbnails) }
        g.async { log(thumbnails) }
    }
}
```

## Takeaways

- Structured concurrency brings the benefits of structured programming (scoping, predictable lifetimes, automatic cleanup) to concurrent Swift code via task trees.
- `async let` and task groups are the two structured forms; prefer them because cancellation and error propagation are automatic.
- Use unstructured `Task { }` when you need to break out of the scope, but manually manage the task handle for cancellation.
- Task cancellation is cooperative — always check `Task.isCancelled` or `Task.checkCancellation()` in long-running work.

---
_Source: WWDC21 Session 10134 page (abstract, chapter summaries, code samples, and resource links)._
