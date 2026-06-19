# Code-Along: Elevate an App with Swift Concurrency
**WWDC25 · Session 270** · [Watch](https://developer.apple.com/videos/play/wwdc2025/270/)

_Platforms:_ iOS 26, macOS Tahoe 26, Swift 6.1

## Overview
This code-along session progressively refactors a working but blocking SwiftUI app to use modern Swift concurrency. Starting from a synchronous implementation that hangs the main thread, the session introduces `async`/`await`, the `.task` modifier, `@concurrent`, `nonisolated`, `async let`, and `withTaskGroup` — each solving a specific performance problem visible in Instruments' Time Profiler.

The session builds intuition for when to use each concurrency tool, explains the Swift 6 data-race safety model, and demonstrates how to track down hangs using Instruments. The example app uses PhotosUI and AVFoundation to process selected photos.

## Key Topics

### Identifying Hangs with Instruments
The Time Profiler instrument reveals main-thread blocking. Any synchronous work on `@MainActor` — including disk I/O, image decoding, and network calls — causes visible hitches. The session uses Instruments to prove the problem before each fix.

### async/await Basics
The `.task` modifier in SwiftUI starts a structured concurrency task tied to the view's lifetime. Suspending with `await` yields the main thread without blocking. The session upgrades synchronous method calls to their async equivalents in `PhotosPickerItem.loadTransferable(type:)` and `Transferable`.

### @concurrent and nonisolated
The **[NEW]** `@concurrent` attribute on a function always executes it on a background thread, even when called from a `@MainActor` context. Previously this required manual `Task.detached`. **[NEW]** `nonisolated` applied to a type in Swift 6.1 makes all of its members nonisolated by default, avoiding boilerplate `nonisolated` annotations on every method.

### async let for Parallelism
`async let` starts a child task immediately and defers binding until the `await` point. This overlaps independent work (e.g., processing multiple images in parallel) within a structured concurrency scope without spawning a TaskGroup.

### withTaskGroup for Dynamic Concurrency
`withTaskGroup` / `TaskGroup` handles a dynamically-sized collection of concurrent tasks. The session iterates a collection of photos, calling `group.addTask { }` for each, then collects results with `for await` iteration over the group.

### @MainActor and Data-Race Safety
Swift 6 enforces data-race safety at compile time. All `@Observable` models are `@MainActor`-isolated by default. The session shows how to correctly pass `Sendable`-typed values across task boundaries and use `@Sendable` closures.

## APIs & Frameworks

**Swift Concurrency (Swift 6.1 / Xcode 26)**
- `async`/`await` — fundamental suspension points
- `.task` modifier (SwiftUI) — structured task tied to view lifetime
- **[NEW]** `@concurrent` function attribute — always executes on background thread
- **[NEW]** `nonisolated` on type — all members become nonisolated (Swift 6.1)
- `async let` — parallel child tasks in structured concurrency
- `withTaskGroup(_:body:)` — dynamic-count concurrent task group
- `TaskGroup.addTask(_:)` — add a task to the group
- `for await in group` — iterate results as tasks complete
- `@MainActor` — main-actor isolation annotation
- `@Sendable` — closure or type that is safe to cross isolation boundaries
- `Task.detached` — unstructured background task (shown as pre-`@concurrent` pattern)

**PhotosUI**
- `PhotosPickerItem` — selected photo item from `PhotosPicker`
- `PhotosPickerItem.loadTransferable(type:)` — async load of the selected asset

**Instruments (Xcode 26)**
- Time Profiler — identifies main-thread hangs; key tool for concurrency auditing

## Code Highlights
Use `@concurrent` to offload work:
```swift
@concurrent
func processImage(_ item: PhotosPickerItem) async throws -> UIImage {
    let data = try await item.loadTransferable(type: Data.self)!
    return try decodeAndResize(data)
}
```

Parallelize with `withTaskGroup`:
```swift
let images = await withTaskGroup(of: UIImage?.self) { group in
    for item in selectedItems {
        group.addTask { try? await processImage(item) }
    }
    var results: [UIImage] = []
    for await image in group {
        if let image { results.append(image) }
    }
    return results
}
```

Make a type's members nonisolated by default (Swift 6.1):
```swift
nonisolated struct ImageProcessor {
    func process(_ data: Data) -> UIImage { ... }
}
```

## Takeaways
- Use Instruments' Time Profiler to prove main-thread blocking before and after each concurrency change.
- Prefer `@concurrent` over `Task.detached` to move synchronous work off the main actor cleanly and with less boilerplate.
- Use `async let` for a fixed number of parallel tasks; use `withTaskGroup` when the number of tasks is determined at runtime.
- Apply `nonisolated` to a type (Swift 6.1) rather than individual methods to avoid repetitive annotations on pure data-processing types.

---
_Source: WWDC25 Session 270 page (abstract, chapter summaries, code samples, and resource links)._
