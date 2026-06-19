# Embracing Swift Concurrency
**WWDC25 · Session 268** · [Watch](https://developer.apple.com/videos/play/wwdc2025/268/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, watchOS 26, visionOS 26

## Overview
This session is the foundational introduction to Swift concurrency, walking through the progressive journey from a fully single-threaded app to one that uses async/await, concurrent tasks, and actors. It explains the mental model behind each step — single-threaded by default, asynchronous tasks for latency hiding, concurrency for background offload, and actors for data isolation — and shows when to take each step and why.

The session introduces the "Approachable Concurrency" and "default actor isolation: main actor" build settings new in Xcode 26, which are the recommended starting configuration for new projects.

## Key Topics

### Step 1: Single-Threaded Code
- All code starts on the **main thread**; that is the right default and can carry apps far.
- The main thread and its data are represented by the **main actor**.
- **Main actor mode** (build setting: `Default Actor Isolation = MainActor`) — **[NEW default in Xcode 26]** — implicitly annotates all types in a module with `@MainActor`, eliminating the need for explicit annotations until concurrency is needed.
- Main actor mode is enabled by default for new app projects created in Xcode 26.

### Step 2: Asynchronous Tasks (async/await)
- Use `async`/`await` to make long-latency operations (network requests, file I/O) non-blocking without moving to a background thread.
- `await` marks a **suspension point** — the function pauses and lets other work run, then resumes when the awaited value is ready.
- Many system APIs (e.g., `URLSession.data(from:)`) are already `async` and handle background network work for you.
- Multiple independent `Task {}` blocks interleave on the main thread — one task suspends while another runs, improving throughput without true concurrency.

### Step 3: Concurrent Code (Background Threads)
- **`@concurrent`** — **[NEW attribute]** apply to a function to tell Swift to always run it on a background thread rather than on whichever actor called it.
- Moving a function to `@concurrent` triggers compiler errors wherever it accesses `@MainActor`-isolated state — these errors identify exactly what needs to be restructured.
- **Strategies for breaking main-actor ties**:
  1. Move the main-actor access to the caller (runs synchronously before the suspension point).
  2. Use `await` to access the main actor asynchronously from the concurrent function.
  3. Mark the code `nonisolated` if it genuinely doesn't need actor protection.
- **`nonisolated`** — the function runs on whatever actor/thread the caller is on; good for library APIs where the caller decides whether to offload.
- The **concurrent thread pool** — all background threads. The system schedules tasks on available threads; when a task suspends, its thread picks up other ready tasks. The app does not control which thread runs a given task.

### Step 4: Sharing Data Safely (Sendable)
- Data crossing actor/thread boundaries must be **`Sendable`** — safe to share concurrently.
- **Value types** (structs, enums, basic types like `String`, `Int`, `URL`, `Data`) are always `Sendable` — each copy is independent.
- **Collections** (`Array`, `Dictionary`) are `Sendable` when their elements are.
- **Main-actor classes** are implicitly `Sendable` because the actor serializes access.
- **Non-Sendable classes** — reference types with mutable state; safe to transfer from one task to another only if all modifications are done before transferring.
- **Closures** that capture non-Sendable state must not be marked `Sendable`; use them from one task at a time.
- The compiler emits errors at every point where non-Sendable data would cross a concurrency boundary — this is compile-time data-race safety.

### Step 5: Actor Types (Off-Main Data)
- **`actor`** — Swift type that serializes access to its own data; similar to a `@MainActor` class but independent of the main thread.
- Actors are `Sendable`; there can be many actor instances.
- Accessing an actor's data from outside requires `await` (hop to the actor's isolation context).
- Use actors when main-actor state is causing too many threads to "check in" with the main thread — move that subsystem's data into a dedicated actor.
- **Do not make UI-facing model classes into actors** — they should stay on the main actor or remain non-Sendable to prevent concurrent mutation.

### Recommended Build Settings (Xcode 26)
- **Approachable Concurrency** — enables a suite of upcoming concurrency language features; recommended for all projects.
- **Default Actor Isolation: MainActor** — recommended for app modules and UI-focused modules; means all code is `@MainActor` by default, single-threaded until you opt into concurrency.

## APIs & Frameworks

### Swift Concurrency Language Features
- `async` / `await` — mark functions as asynchronous and suspension points.
- `Task { }` — create a new asynchronous task.
- `@MainActor` — global actor annotation for main-thread isolation.
- `@concurrent` — **[NEW]** attribute to force a function off any actor onto the concurrent thread pool.
- `nonisolated` — function runs on whatever isolation context calls it.
- `actor` type declaration — custom actor with isolated data.
- `Sendable` protocol — marks types safe for concurrent sharing.
- `@Sendable` — marks a closure as safe to pass across concurrency boundaries.

### Xcode Build Settings (NEW defaults)
- `SWIFT_DEFAULT_ACTOR_ISOLATION = MainActor` — implicit `@MainActor` for entire module.
- Approachable Concurrency setting — enables upcoming concurrency language features.

### Resources
- Swift Migration Guide (swift.org) — migration to Swift 6 data-race safety.
- The Swift Programming Language: Concurrency (docs.swift.org).

## Code Highlights

```swift
// Make a function async to hide network latency
func fetchAndDisplayImage(url: URL) async throws {
    let (data, _) = try await URLSession.shared.data(from: url)
    let image = decodeImage(data)
    view.displayImage(image)
}
```

```swift
// Offload decoding to a background thread
@concurrent
func decodeImage(_ data: Data) -> Image {
    // runs on the concurrent thread pool
    Image(data: data)
}

// Caller awaits the result
func fetchAndDisplayImage(url: URL) async throws {
    if let cached = imageCache[url] { view.displayImage(cached); return }
    let (data, _) = try await URLSession.shared.data(from: url)
    let image = await decodeImage(data)  // runs off-main
    view.displayImage(image)
}
```

```swift
// Custom actor for off-main subsystem
actor NetworkManager {
    var openConnections: [URL: Connection] = [:]
    func fetchData(from url: URL) async throws -> Data { ... }
}
```

## Takeaways
- Start every project single-threaded; only introduce async/await and concurrency when profiling shows it is necessary.
- Enable "Default Actor Isolation: MainActor" and "Approachable Concurrency" build settings in Xcode 26 for all new app projects — this is the recommended path.
- Use `@concurrent` to move compute-heavy functions off the main thread; let the compiler guide you to the exact data-sharing violations that need fixing.
- Keep model classes `@MainActor` and non-`Sendable`; only introduce actor types when the main actor becomes a bottleneck due to many subsystems checking in from background tasks.

---
_Source: WWDC25 Session 268 page (abstract, chapters, full transcript, and code samples)._
