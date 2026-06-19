# What's New in Swift
**WWDC21 · Session 10192** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10192/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
Swift 5.5 is the biggest release since Swift 1.0, headlined by Swift Concurrency: `async`/`await`, structured concurrency with `async let`, and the `actor` type. These three interlocking features provide a safe, efficient, and ergonomic model for asynchronous and concurrent programming that eliminates callback pyramids, enforces structured lifetimes for parallel tasks, and prevents data races at the language level.

Beyond concurrency, Swift 5.5 brings a sweeping set of ergonomic improvements (Result Builders, Codable enum synthesis, property wrappers on function parameters, `#if` postfix expressions, `CGFloat`/`Double` interop), major build performance wins (incremental imports, module dependency graph pre-computation), and a new family of open-source Swift packages (Swift Collections, Algorithms, System, Numerics, ArgumentParser). DocC, a documentation compiler integrated into Xcode 13, is also announced and will be open-sourced.

## Key Topics

### Swift Concurrency: async/await
`async` marks a function as suspendable; `await` marks a call site where the caller may be suspended. Suspended functions do not block their thread — the runtime reuses the thread for other work. Error handling integrates naturally with `try`/`catch`. The Swift Driver itself (the first part of the compiler written in Swift) was necessary to implement this correctly.

### Structured Concurrency with async let
`async let` initiates a child task in parallel. The parent task is automatically suspended at the first `await` on an `async let` variable. Critically, child tasks cannot outlive their parent — the function will not return until all child tasks finish, even if an error is thrown. The runtime signals unfinished tasks to cancel early on error.

### Actors
The `actor` keyword creates a reference type that protects its mutable state from concurrent access. External callers must `await` calls into an actor to ensure the runtime can serialize access. Actors integrate with `async`/`await` so methods on an actor can themselves be `async`, suspending while waiting on network or other actors without blocking other actor methods.

### Ergonomic Language Improvements (SE proposals)
- **Result Builders** (SE-0289) — standardized and refined; the foundation of SwiftUI's DSL
- **Codable enum synthesis** — compiler now auto-synthesizes `Codable` for enums with associated values
- **Static member lookup in generic contexts** (SE-0299) — dot-notation (`.large`, `.regular`) works for protocol static properties, not just enums
- **Property wrappers on function/closure parameters** (SE-0293) — `@propertyWrapper` structs can now annotate parameters
- **`#if` for postfix expressions** — `#if` can wrap modifier chains and postfix expressions
- **`CGFloat`/`Double` interoperability** — compiler transparently converts between these types at Apple API boundaries

### Build Performance
- **Incremental imports** — source files are no longer all recompiled when an imported module changes; average recompilation drops to <1/10 of previous, build time cut by ~1/3
- **Module dependency graph pre-computation** — up-front graph calculation enables faster incremental builds
- **Incremental recompilation for extensions** — changing an extension body no longer causes cascading recompilation

### DocC — Documentation Compiler
Integrated into Xcode 13. Markdown comments in Swift source become structured, browsable documentation. Will be open-sourced. Covered in depth in four companion sessions.

### ARC Optimization
New compiler reference-tracking reduces the number of retain/release operations inserted by the compiler. An opt-in Xcode setting ("Optimize Object Lifetimes") exposes more aggressive ARC optimization.

### Swift Open-Source Packages

**Swift Collections** — `Deque` (efficient insertion/removal at both ends), `OrderedSet` (ordered, unique, random-access), `OrderedDictionary` (ordered key-value pairs).

**Swift Algorithms** — 40+ Sequence/Collection algorithms: combinations, permutations, chunked iteration, smallest/largest N elements, random sampling.

**Swift System** — Low-level system call interfaces. New `FilePath` APIs: query/set extensions, add/remove components, path normalization, `ComponentView` collection, full Windows path support.

**Swift Numerics** — `Float16` on Apple Silicon Macs; `Float16` complex numbers; complex number support for all elementary functions (log, sin, cos, etc.).

**Swift ArgumentParser** — Fish shell completion scripts, joined short options, improved error messages; adopted by Swift Package Manager itself in Xcode 12.5.

### Swift Package Collections
Curated JSON lists of packages publishable anywhere. Browsable and searchable in Xcode 13's new package search screen. Xcode pre-wired with the Apple package collection. Import-completion integration: Xcode suggests adding a package from collections when an import cannot be resolved.

### Server/Linux Improvements
- Static linking on Linux (faster startup, single-file deployment)
- JSON encoding/decoding reimplemented on Linux (performance gains)
- AWS Lambda runtime refactored to use async/await; 33% faster cold start, 40% faster API Gateway invocation

## APIs & Frameworks

- `async` function modifier **[NEW]** — marks a function as suspendable
- `await` expression **[NEW]** — marks a potential suspension point when calling an async function
- `async let` **[NEW]** — creates a child task running in parallel; awaited lazily
- `actor` type **[NEW]** — reference type with automatic mutual exclusion on mutable state
- `Task` **[NEW]** — unstructured concurrency handle
- `TaskGroup` **[NEW]** — dynamic child task creation via `withTaskGroup`
- `@MainActor` **[NEW]** — global actor for main-thread-bound code
- `AsyncSequence` **[NEW]** — protocol for async iteration with `for await in`
- `Sendable` protocol **[NEW]** — marks types safe to transfer across concurrency domains
- `@Sendable` **[NEW]** — closure attribute for concurrency-safe closures
- `@resultBuilder` (SE-0289) **[NEW]** — attribute formalizing Result Builder DSL syntax
- `Codable` enum auto-synthesis **[NEW]** — no manual boilerplate for enums with associated values
- Property wrappers on function parameters (SE-0293) **[NEW]**
- `CGFloat`/`Double` implicit conversion **[NEW]**
- `DocC` — documentation compiler in Xcode 13 **[NEW]**
- `Swift Collections` package — `Deque`, `OrderedSet`, `OrderedDictionary` **[NEW]**
- `Swift Algorithms` package — 40+ sequence/collection algorithms **[NEW]**
- `Swift System` `FilePath` extensions — `extension`, `stem`, `components`, `ComponentView` **[NEW]**
- `Swift Numerics` `Float16` + complex numbers **[NEW]**
- Swift Package Collections — JSON-based curated package lists **[NEW]**
- `URLSession.data(from:delegate:)` async overload **[NEW]** — replaces `dataTask(with:completionHandler:)`

## Code Highlights

async/await replacing completion handlers:
```swift
func fetchImage(id: String) async throws -> UIImage {
    let (data, _) = try await URLSession.shared.data(from: imageURL(id))
    return UIImage(data: data)!
}
```

Structured concurrency with async let:
```swift
func renderCombinedImage() async throws -> Image {
    async let background = renderBackground()
    async let foreground = renderForeground()
    let title = try await renderTitle()   // runs sequentially; bg/fg run in parallel
    return try await merge(background, foreground, title)
}
```

Actor protecting shared state:
```swift
actor StatsCounter {
    private var count = 0
    func increment() { count += 1 }
    func publish() async { await upload(count) }
}
// External caller:
await counter.increment()
```

Swift Collections — Deque:
```swift
import Collections
var colors: Deque = ["red", "yellow", "blue"]
colors.prepend("green")
colors.popFirst() // "green"
```

## Takeaways

- Swift Concurrency (`async`/`await`, `async let`, `actor`) is the headline feature of Swift 5.5 and fundamentally changes how asynchronous code is written — structured, readable, and race-condition-safe.
- Incremental imports cut build times dramatically for modular projects; upgrading to Xcode 13 delivers this automatically.
- The new open-source Swift packages (Collections, Algorithms, System, Numerics) fill long-standing gaps in the standard library and are production-ready.
- Swift 6 research is already underway to have the compiler statically eliminate entire classes of concurrency bugs.

---
_Source: WWDC21 Session 10192 page (abstract, chapter summaries, code samples, and resource links)._
