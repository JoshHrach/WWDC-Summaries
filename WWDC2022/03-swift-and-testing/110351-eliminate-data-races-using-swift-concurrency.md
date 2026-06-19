# Eliminate Data Races Using Swift Concurrency
**WWDC22 · Session 110351** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110351/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16, watchOS 9

## Overview
Presented by Doug from the Swift team, this session takes a holistic view of Swift Concurrency as a system for eliminating data races — not just a set of individual language features. Rather than revisiting async/await mechanics (covered in 2021 sessions), it addresses the harder questions: how does isolation work across tasks and actors, what does `Sendable` actually enforce, when can an actor still produce high-level data races, and how do you migrate an existing codebase incrementally?

The session uses a vivid nautical analogy (tasks as boats, actors as islands, the Swift compiler as a customs inspector) to make the abstractions concrete. Three major themes emerge: task isolation via `Sendable`, actor isolation and the semantics of `nonisolated` code, and the three-tier strict concurrency checking build settings that ship with Swift 5.7.

## Key Topics

### Task Isolation and Sendable
- Tasks are self-contained: each has its own state, runs sequentially, may suspend at `await` points.
- Value types (structs, enums) copy their data — safe to share between tasks; reference types (classes) share identity — potentially unsafe.
- `Sendable` protocol marks types that are safe to pass across isolation domain boundaries.
  - Structs/enums conforming to `Sendable` require all stored properties to also be `Sendable`; conformance can be inferred by the compiler for non-public types.
  - Collections of `Sendable` types are `Sendable` via conditional conformance.
  - `final class` with only immutable storage can be `Sendable`.
  - Mutable reference types cannot be `Sendable` unless using `@unchecked Sendable` (manual synchronization required).
- `Task<Success: Sendable, Failure: Error>` — generic constraint enforces that task results must be `Sendable`.
- `@Sendable` function type — values of this type conform to `Sendable`; used on closure parameters (e.g., `Task.detached(operation:)`) to enforce that captured state is safe to share.
- The Swift compiler checks `Sendable` conformances at every task boundary (task creation, task result access, actor method calls).

### Actor Isolation
- `actor` keyword — defines a reference type that serializes access to its state; only one task runs on an actor at a time.
- All actors are implicitly `Sendable` — holding a reference to an actor from another isolation domain is safe (like having a map to an island).
- Actor-isolated code: instance properties, instance methods, non-`@Sendable` closures executing within the actor, and `Task { }` initializers (which inherit actor isolation from context).
- Non-isolated code: `Task.detached { }` closures, functions marked `nonisolated`, and synchronous functions called from a non-isolated context.
- `nonisolated func` — explicitly opts a method out of actor isolation; must use `await` to access actor-isolated state.
- Non-isolated async code runs on the **global cooperative pool**, not on any actor.
- Non-`Sendable` types cannot be passed into or out of an actor — compiler enforces this at await boundaries.

### The Main Actor
- `@MainActor` — special actor representing the main thread; used for all UI work.
- Apply to a function/closure to require it runs on the main thread.
- Apply to a class to make all properties and methods main-actor-isolated (suitable for `UIViewController`, `NSViewController`, SwiftUI view models).
- `@MainActor` class instances are `Sendable` — safe to reference from other tasks/actors.
- Architectural implication: views and view controllers live on `@MainActor`; other model/business logic should use dedicated actors or independent tasks.

### Atomicity and High-Level Data Races
- Actors eliminate *low-level* data races (data corruption from concurrent access), but not *high-level* data races (unexpected state changes between two `await` points).
- Example: reading actor state, computing a new value in non-isolated async context, then writing back — another task may modify the actor between the two `await` calls.
- Fix: move multi-step state mutations into a single synchronous actor method (no `await` inside), which runs atomically on the actor.
- Rule: each synchronous actor function should leave the actor in a consistent state; async actor functions should verify invariants at every `await`.

### Ordering and AsyncStream
- Actors are **not** FIFO queues; they process the highest-priority pending task first (no priority inversion).
- This differs from serial `DispatchQueue` (always FIFO).
- Use `AsyncStream` to order events: one task iterates with `for await event in stream`; producers append to the stream, preserving order.

### Strict Concurrency Checking (Swift 5.7) **[NEW]**
Three build settings (SWIFT_STRICT_CONCURRENCY):
1. **Minimal** (default): diagnoses only explicit `Sendable` conformance errors; matches Swift 5.5/5.6 behavior.
2. **Targeted**: enables `Sendable` checking for code already using `async`/`await`, tasks, or actors; warns on non-`Sendable` captures in newly created tasks.
3. **Complete**: approximates Swift 6 semantics — checks all code in the module, including code using `DispatchQueue` callbacks that take `@Sendable` closures.
- All violations in Swift 5 are **warnings**, not errors, to ease incremental migration.
- `@preconcurrency import ModuleName` — silences `Sendable` warnings for types from modules that haven't adopted strict concurrency yet; warnings return automatically if/when the module is updated.

## APIs & Frameworks

### Swift Standard Library / Concurrency
- `Sendable` protocol — marks types safe for cross-isolation sharing
- `@Sendable` — attribute on function/closure types for `Sendable` function values
- `@unchecked Sendable` — opt out of compiler checking (use with care)
- `Task<Success: Sendable, Failure: Error>` — task generic constraint enforces Sendable result
- `Task.detached(priority:operation:)` — creates independent task (no actor isolation inherited)
- `Task { }` initializer — creates task inheriting current actor isolation
- `actor` keyword — actor type declaration
- `nonisolated` keyword — opt out of actor isolation for specific methods
- `@MainActor` attribute — main thread isolation for types, methods, closures
- `AsyncStream<Element>` — ordered asynchronous sequence for event streams
- `AsyncStream.Continuation` — append elements to an `AsyncStream`
- `withTaskCancellationHandler(operation:onCancel:)` — handle task cancellation
- SWIFT_STRICT_CONCURRENCY build setting **[NEW]** — `minimal`, `targeted`, `complete`
- `@preconcurrency import` **[NEW]** — suppress Sendable warnings for imported types

## Code Highlights

```swift
// Value type is Sendable; reference type is not
struct Pineapple: Sendable {
    var weight: Double
    var ripeness: Ripeness
}
final class Chicken { // cannot be Sendable — mutable stored property
    var currentHunger: HungerLevel
}

// Task result must be Sendable — error if Chicken is returned
let petAdoption = Task {
    let chickens = await hatchNewFlock()
    return chickens.randomElement()! // ERROR: Chicken is not Sendable
}

// Actor: only one task enters at a time
actor Island {
    var flock: [Chicken]
    var food: [Pineapple]

    // Atomic synchronous mutation — no await, invariants preserved
    func deposit(pineapples: [Pineapple]) {
        var food = self.food
        food += pineapples
        self.food = food
    }
}

// nonisolated: runs on global pool, must await to access island state
extension Island {
    nonisolated func meetTheFlock() async {
        let flockNames = await flock.map { $0.name }
        print("Meet our flock: \(flockNames)")
    }
}

// @MainActor class — all properties and methods on main thread
@MainActor
class ChickenValley: Sendable {
    var flock: [Chicken]
    var food: [Pineapple]

    func advanceTime() {
        for chicken in flock { chicken.eat(from: &food) }
    }
}

// AsyncStream for ordered event processing
for await event in eventStream {
    await process(event)
}

// @preconcurrency import for incremental migration
@preconcurrency import FarmAnimals  // silences Sendable warnings for Chicken
```

## Takeaways
- `Sendable` is the key to task isolation: value types get it automatically; mutable reference types don't. The Swift compiler enforces this at every task and actor boundary.
- Actor isolation eliminates low-level data races, but high-level races (reading state, doing async work, writing back) can still occur — avoid them by keeping multi-step mutations in synchronous actor methods.
- Actors are not FIFO queues; use `AsyncStream` when ordering of events is required.
- Migrate incrementally using the three-tier `SWIFT_STRICT_CONCURRENCY` build setting: start with `targeted` to find race conditions in newly concurrent code, then work toward `complete` for full Swift 6 readiness.
- `@preconcurrency import` is a practical escape hatch for third-party modules that haven't adopted `Sendable` yet; the compiler will re-check your assumptions when those modules are updated.

---
_Source: WWDC22 Session 110351 page (transcript, code samples, and resource links)._
