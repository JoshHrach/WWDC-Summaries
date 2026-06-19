# Swift Concurrency: Update a Sample App
**WWDC21 · Session 10194** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10194/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8

## Overview
This code-along session walks through a complete, real-world migration of the "Coffee Tracker" watchOS app (originally from WWDC20) from a completion-handler/DispatchQueue architecture to Swift's new concurrency features: async/await, actors, `@MainActor`, `nonisolated`, and `withCheckedThrowingContinuation`.

The session emphasizes step-by-step, incremental migration — keep the app compiling and running at each stage by adding async alternatives alongside existing completion-handler methods, then deprecating and removing the old versions once callers are updated. It also illustrates how compiler errors from Swift's data-race safety model guide the migration and surface hidden race conditions in existing code.

The app's architecture goes from three ad-hoc concurrent queues (main, background dispatch, arbitrary HealthKit callbacks) to a clean alignment between type architecture and concurrency: SwiftUI views and the data model live on `@MainActor`, HealthKit work lives on a `HealthKitController` actor, and file I/O lives on a private `CoffeeDataStore` actor.

## Key Topics

### Async/Await Migration
Replace SDK completion handlers with their `async throws` equivalents using `try await`. When callers are not yet async, wrap the call in `Task { }` to bridge the sync/async boundary. Use `@available(*, deprecated)` shims that forward completion-handler calls to the new async version so existing callers keep working during incremental migration.

### `withCheckedThrowingContinuation`
Used to bridge callback-based APIs (e.g., `HKAnchoredObjectQuery` with its completion block) into `async throws` functions. The continuation is resumed either with `continuation.resume(throwing:)` on error or `continuation.resume(returning:)` on success.

### `@MainActor`
Annotate a function or entire class with `@MainActor` to have the compiler enforce that all access happens on the main thread. Replaces manual `DispatchQueue.main.async` calls and `assert(Thread.isMainThread)`. Calling a `@MainActor` function from a non-main context requires `await`. SwiftUI views that use `@EnvironmentObject` or `@ObservedObject` are automatically `@MainActor`.

### `MainActor.run { }`
Groups multiple main-actor operations atomically within a single suspension point, preventing the main run loop from interleaving between them. Useful when multiple UI updates must happen together without any intermediate suspensions.

### Actors for Background Isolation
Convert a class with a `DispatchQueue` for background work into an `actor`. The actor's stored properties are automatically isolated — access from outside requires `await`. Use `nonisolated` on methods that don't touch isolated state (e.g., deprecated shims that only spin up a `Task`) so they can be called without `await`.

### Avoiding `didSet` for Async Side-Effects
`didSet` on a property cannot be `async`. Removing `didSet` and replacing it with an explicit `async` function `drinksUpdated()` — called at every mutation site — keeps async code structured and avoids hidden synchronous I/O on the main thread.

### Incremental Migration Strategy
1. Convert leaf-level async SDK calls first.
2. Add async alternatives with `@available(*, deprecated)` for existing callers.
3. Work up the call chain following deprecation warnings.
4. Convert types to actors last, after their methods are already async.
5. Address compiler errors one layer at a time; avoid cascading changes.

## APIs & Frameworks

**Swift Concurrency (all new in Swift 5.5)**
- `async` / `await` **[NEW]** — marks and calls asynchronous functions
- `Task { }` **[NEW]** — creates an unstructured async task from synchronous context; inherits actor context of caller
- `Task.detached { }` **[NEW]** — creates an unstructured async task that does not inherit caller's actor context
- `actor` keyword **[NEW]** — creates a reference type with actor-isolated state
- `@MainActor` **[NEW]** — global actor annotation for main-thread isolation; applies to functions, properties, or entire types
- `nonisolated` **[NEW]** — opts a method out of actor isolation so it can be called from any context
- `MainActor.run { }` **[NEW]** — runs a closure synchronously on the main actor from an async context
- `withCheckedThrowingContinuation(_:)` **[NEW]** — bridges callback-based APIs into async/await
- `withCheckedContinuation(_:)` **[NEW]** — non-throwing variant of continuation bridging
- `CheckedContinuation.resume(returning:)` / `resume(throwing:)` **[NEW]**

**HealthKit**
- `HKHealthStore.save(_:)` — async variant **[NEW]**; previously required completion handler
- `HKHealthStore.requestAuthorization(toShare:read:)` — async variant **[NEW]**
- `HKAnchoredObjectQuery` — bridged to async via `withCheckedThrowingContinuation`

**SwiftUI / ObservableObject**
- `ObservableObject` — `@Published` properties must only be mutated on main thread; solved by `@MainActor` on the class
- `@Published` — annotated properties trigger UI updates
- `@EnvironmentObject` / `@ObservedObject` — accessing these in a SwiftUI view infers `@MainActor` on the view

**WatchKit**
- `WKExtensionDelegate.handle(_:)` — runs on `@MainActor`; safe to call main-actor methods directly

**Foundation / FileManager**
- `PropertyListEncoder` / `PropertyListDecoder` — used for local persistence in the `CoffeeDataStore` actor

## Code Highlights

Continuation bridge for HealthKit query:
```swift
private func queryHealthKit() async throws -> ([HKSample]?, [HKDeletedObject]?, HKQueryAnchor?) {
    return try await withCheckedThrowingContinuation { continuation in
        let query = HKAnchoredObjectQuery(type: caffeineType, ...) { _, samples, deleted, anchor, error in
            if let error = error {
                continuation.resume(throwing: error)
            } else {
                continuation.resume(returning: (samples, deleted, anchor))
            }
        }
        store.execute(query)
    }
}
```

Deprecated shim forwarding to async alternative:
```swift
@available(*, deprecated, message: "Prefer async alternative instead")
nonisolated public func requestAuthorization(completionHandler: @escaping (Bool) -> Void) {
    Task { completionHandler(await requestAuthorization()) }
}
```

Actor for file I/O, `@MainActor` for UI model:
```swift
private actor CoffeeDataStore { /* load/save */ }

@MainActor
class CoffeeData: ObservableObject {
    private let store = CoffeeDataStore()
    @Published public private(set) var currentDrinks: [Drink] = []
}
```

## Takeaways
- Migrate incrementally: convert leaf async calls first, add deprecated shims, follow deprecation warnings up the call chain, and convert types to actors last.
- `withCheckedThrowingContinuation` is the standard bridge for any callback-based API that cannot yet provide an async variant.
- Annotating an `ObservableObject` class with `@MainActor` is the correct pattern for SwiftUI view models — it enforces main-thread mutation without manual dispatch calls.
- Compiler errors from actor isolation are intentional guides: they expose hidden race conditions and indicate exactly which access patterns need restructuring.

---
_Source: WWDC21 Session 10194 page (abstract, chapter summaries, code samples, and resource links)._
