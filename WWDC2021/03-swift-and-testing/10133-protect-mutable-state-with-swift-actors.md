# Protect Mutable State with Swift Actors
**WWDC21 · Session 10133** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10133/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
Data races — two threads concurrently accessing the same mutable state where at least one is a write — are trivial to create but notoriously hard to debug. This session introduces Swift actors, a new reference type that provides mutual exclusion over mutable state as a language-level guarantee rather than a developer discipline.

The session walks through why value semantics alone cannot always eliminate the need for shared mutable state, then explains how the `actor` keyword declares a type whose internal state is isolated and whose external callers must use `await`. Key pitfalls like actor reentrancy are examined in depth with a realistic image-downloader example.

The final third covers the `Sendable` protocol for type-safe concurrency boundaries, `@Sendable` closures, and the special `@MainActor` global actor that replaces `DispatchQueue.main` patterns.

## Key Topics

### Actors and Actor Isolation
- `actor` is a new named type in Swift, similar to a class but with automatic mutual exclusion over instance state.
- External callers must use `await` to cross the actor boundary; the call suspends if the actor is busy.
- Synchronous code within an actor runs uninterrupted to completion — no `await` needed for internal calls.
- `nonisolated` keyword opts a method out of actor isolation, allowing it to satisfy synchronous protocol requirements while only accessing immutable state.

### Actor Reentrancy
- `await` inside an actor is a potential suspension point; the world (and actor state) can change before resumption.
- Design for reentrancy by checking assumptions after every `await`; prefer encapsulating state mutations in synchronous functions.
- The `ImageDownloader` pattern: use a `Task`-wrapping cache entry (`case inProgress(Task<Image, Error>)`) to avoid redundant downloads and stale cache writes.

### Sendable Types
- `Sendable` protocol marks types safe to share across actor boundaries.
- Value types (structs, enums) are generally Sendable; actor types are Sendable; most classes are not.
- Conditional conformance (`extension Pair: Sendable where T: Sendable, U: Sendable`) propagates Sendable through generic types.
- `@Sendable` function/closure types cannot capture mutable local variables or be actor-isolated, preventing data races via closures.

### The Main Actor
- `@MainActor` is a global actor backed by the main dispatch queue — interchangeable at runtime with `DispatchQueue.main`.
- Mark functions or entire types with `@MainActor` to guarantee they always execute on the main thread, eliminating manual `DispatchQueue.main.async` calls.
- `nonisolated` can still opt individual methods off the main actor.

## APIs & Frameworks

- `actor` keyword **[NEW]** — new type declaration for actors
- `nonisolated` keyword **[NEW]** — opts a declaration out of actor isolation
- `@MainActor` attribute **[NEW]** — global actor representing the main thread
- `Sendable` protocol **[NEW]** — marks a type safe to share across concurrency domains
- `@Sendable` function/closure attribute **[NEW]** — marks a closure as safe to pass across actors
- `Task.detached { }` **[NEW]** — creates a detached (unstructured) concurrent task
- `Task<Success, Failure>` **[NEW]** — represents an asynchronous operation
- `task.value` **[NEW]** — `async` property to await a `Task`'s result
- `await` keyword **[NEW in structured concurrency context]** — suspends caller at an actor boundary or async call
- `async`/`throws` function modifiers
- `DispatchQueue.main` (superseded by `@MainActor` in modern Swift)

## Code Highlights

Declaring and calling an actor:
```swift
actor Counter {
    var value = 0
    func increment() -> Int {
        value = value + 1
        return value
    }
}
// External call requires await:
print(await counter.increment())
```

Actor reentrancy fix — check cache after `await`:
```swift
let image = try await downloadImage(from: url)
cache[url] = cache[url, default: image]  // keep original if already cached
return cache[url]
```

Marking a type as main-actor-isolated:
```swift
@MainActor class MyViewController: UIViewController {
    func onPress(...) { ... } // implicitly @MainActor
    nonisolated func fetchLatestAndDisplay() async { ... }
}
```

Sendable conditional conformance:
```swift
extension Pair: Sendable where T: Sendable, U: Sendable {}
```

## Takeaways
- Actors provide language-enforced mutual exclusion; you can never forget to synchronize because the compiler enforces it.
- Every `await` inside an actor is a potential reentrancy point — always re-check state after an `await` rather than carrying assumptions forward.
- Adopt `Sendable` on your value types now; Swift will eventually enforce Sendable boundaries at actor crossings statically.
- Replace `DispatchQueue.main.async` patterns with `@MainActor` annotations for clearer, compiler-verified main-thread guarantees.

---
_Source: WWDC21 Session 10133 page (abstract, chapter summaries, code samples, and resource links)._
