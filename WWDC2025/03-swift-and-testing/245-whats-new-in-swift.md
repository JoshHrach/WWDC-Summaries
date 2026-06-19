# What's new in Swift
**WWDC25 · Session 245** · [Watch](https://developer.apple.com/videos/play/wwdc2025/245/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26, watchOS 26, Linux, Windows, FreeBSD, WebAssembly

## Overview
Swift 6.2 focuses on three goals: better development workflows (faster builds, improved diagnostics, richer debugging), new and modernized library APIs (Subprocess, Foundation notifications, Observation AsyncSequence, Swift Testing), and language evolution that makes concurrency more approachable while unlocking peak performance (InlineArray, Span, main-actor-by-default mode, isolated conformances, `@concurrent`).

The session also covers Swift's expanding reach: Embedded Swift in iPhone coprocessors, strict memory safety mode adopted in WebKit and Messages, Swift server adoption (40% throughput gain vs. Java), Java interoperability via swift-java, a new containerization library, and new platform targets including FreeBSD and WebAssembly.

## Key Topics

### Workflow: swiftlang and tooling
The swiftlang GitHub organization now has 50+ projects. **Swift Build** (Xcode's build engine) is open-sourced and being integrated into Swift Package Manager for a unified build experience across Xcode and Swift.org toolchains. **swiftly** — a Swift toolchain version manager originally for Linux — now supports macOS (v1.0 on swift.org); Xcode shows toolchains installed by swiftly in its Toolchains menu. The VS Code Swift extension is now officially verified and distributed by Swift.org and gained: background indexing, improved code completion, automatic LLDB support, a project panel, and live DocC previews.

### Workflow: Building
**Pre-built swift-syntax**: Swift PM and Xcode support pre-built swift-syntax dependencies for packages that provide macros, eliminating a costly clean-build step (minutes saved on some projects). **Diagnostic documentation**: Extended explanations for common concurrency and other errors, accessible from IDEs and swift.org. **Warning control**: New fine-grained control over which warnings are treated as errors (e.g., treat all warnings as errors but exempt deprecated API warnings).

### Workflow: Debugging
**Async step-through**: LLDB now follows execution into async functions across thread hops. Task IDs visible in backtrace and Program Counter views; `swift task info` command shows priority and child tasks. Task names appear in Instruments Swift Concurrency template profiles. **Faster debugger**: Explicitly Built Modules (default in Xcode 26) allow the debugger to reuse build-time module graphs, making the first `p`/`po` command much faster.

### Libraries: Subprocess
New **`Subprocess`** package (v0.1) for launching and managing subprocesses from Swift. `run(.name("pwd"))` or `run(.path(swiftPath), arguments: [...])` — async, returns exit status, standard output, error. Fine-grained control over process execution and platform-specific configuration.

### Libraries: Foundation — typed notifications
`NotificationCenter` now supports **concrete notification types**. `center.addObserver(of: screen, for: .keyboardWillShow) { keyboardState in ... }` — typed payload, no `userInfo` dictionary, no casting. Two protocol markers: `NotificationCenter.MainActorMessage` (posted synchronously on main actor — safe to access main-actor APIs in handler) and `NotificationCenter.AsyncMessage` (posted asynchronously on arbitrary thread). SDK frameworks (UIKit, Foundation) provide concrete types; apps can add custom conformances.

### Libraries: Observation — AsyncSequence streaming
New **`Observations`** type: create with a closure that computes a value from `@Observable` properties; returns an `AsyncSequence` of updated values. Updates are transactional — all synchronous changes to multiple observed properties between a `willSet` and the next `await` produce a single update. Iterate with `for await value in values { ... }`.

### Libraries: Swift Testing
Two new capabilities in Swift Testing: **custom attachments** — `Attachment.record(data, named:)` to attach `Data`, `String`, or custom `Attachable`-conforming types to test results for failure diagnosis; **exit tests** — `await #expect(processExitsWith: .failure) { ... }` or `#require(processExitsWith:)` to validate code that calls `precondition`/`fatalError`.

### Performance: InlineArray and Span
**`InlineArray<N, Element>`** — fixed-size array with inline storage (no heap allocation). Size is part of the type (integer type parameter, a new generics feature). Inferred from array literals. Supports copyable and non-copyable elements. Enables bounds-check elimination when index is statically known.

**`Span`** — memory-safe abstraction for direct access to contiguous storage without unsafe pointers. Provided by `Array`, `ArraySlice`, `InlineArray`, and other containers via a `.span` property. Lifetime-dependent on the original container (compile-time enforcement, no runtime overhead). Prevents use-after-free and overlapping modification.

### Concurrency: approachability improvements
Swift 6.2 changes philosophy from "eagerly offload async work to the background" to **"stay on the caller's actor by default"**:

1. **Async functions run on caller's actor**: Non-actor-tied async functions no longer hop to the generic executor — they stay on whichever actor called them, eliminating accidental data races.
2. **Isolated conformances**: Protocol conformances can be annotated with an actor (`extension StickerModel: @MainActor Exportable`) — compiler enforces that the conformance is only used on that actor.
3. **Main-actor-by-default mode** (opt-in): All code in a module/file runs on the main actor by default. Eliminates boilerplate `@MainActor` annotations for apps and scripts. Enable in Xcode build settings under Swift Compiler — Concurrency, or via `SwiftSettings` in package manifests.
4. **`@concurrent` attribute**: Mark specific functions to always run on the concurrent thread pool, explicitly opting into background execution for CPU-intensive work.

Migration tooling available at swift.org/migration to apply these changes automatically.

## APIs & Frameworks

### Swift Standard Library
- **`InlineArray<N, Element>`** **[NEW]** — fixed-size inline array; `InlineArray<3, Int>` inferred from `[1, 2, 3]`
- **`Span`** **[NEW]** — lifetime-safe view over contiguous memory; `.span` property on `Array`, `ArraySlice`, `InlineArray`
- Integer type parameters (new generics feature) **[NEW]** — enables `InlineArray` size in type signature

### Subprocess package
- **`import Subprocess`** **[NEW]** — new package (v0.1)
- **`run(.name(_:))`** **[NEW]** — launch by `$PATH` lookup
- **`run(.path(_:), arguments:)`** **[NEW]** — launch by `FilePath`
- `result.standardOutput`, `result.exitStatus` **[NEW]**

### Foundation / Observation
- **`NotificationCenter.MainActorMessage`** protocol **[NEW]** — main-actor-posted notifications
- **`NotificationCenter.AsyncMessage`** protocol **[NEW]** — async-posted notifications
- `NotificationCenter.addObserver(of:for:_:)` **[NEW]** — typed handler
- **`Observations`** type **[NEW]** — `init(_ body:)` returns `AsyncSequence` of observed values
- `@Observable` macro (existing) — expanded with `Observations` streaming

### Swift Testing
- **`Attachment.record(_:named:)`** **[NEW]** — attach `Data`/`String`/`Attachable` types
- **`Attachable`** protocol **[NEW]** — implement for custom attachment types
- **`#expect(processExitsWith:)`** **[NEW]** — exit test macro
- **`#require(processExitsWith:)`** **[NEW]** — exit test requiring macro

### Language features
- **`@concurrent`** **[NEW]** attribute — force function to run on concurrent executor
- **Isolated conformances** **[NEW]** — `extension Type: @ActorType Protocol`
- **Main-actor-by-default mode** **[NEW]** — `SwiftSetting.defaultIsolation(.mainActor)`
- **Strict memory safety mode** **[NEW]** — opt-in compiler flag; requires explicit `unsafe` annotations
- **Integer type parameters** **[NEW]** — integers as generic type arguments

### Platforms (new)
- FreeBSD **[NEW]** — official support
- WebAssembly **[NEW]** — official support in progress; preview available

## Code Highlights

```swift
// Typed notification observer
let token = NotificationCenter.default.addObserver(of: screen, for: .keyboardWillShow) { state in
    animate(withDuration: state.animationDuration) {
        bottomConstraint.constant = view.bounds.maxY - state.endFrame.minY
    }
}

// Observations AsyncSequence
let values = Observations {
    "\(player.score) points and \(player.item)"
}
for await value in values { print(value) }

// InlineArray — no heap allocation
var buffer: InlineArray<16, UInt8> = .init(repeating: 0)

// Span — safe memory access
let span = myArray.span  // Lifetime bound to myArray
processSpan(span)

// Main-actor-by-default in SwiftSettings
.enableExperimentalFeature("DefaultIsolation(MainActor)")

// @concurrent offloads to background
@concurrent
static func extractSubject(from data: Data) async -> Sticker { ... }

// Isolated conformance
extension StickerModel: @MainActor Exportable {
    func export() { photoProcessor.exportAsPNG() }
}

// Exit test
await #expect(processExitsWith: .failure) {
    _ = Proposal(id: "SE-NNNN").number  // triggers precondition
}
```

## Takeaways
- Enable main-actor-by-default mode for new app/script targets — it eliminates most `@MainActor` annotation noise and makes Swift 6 data-race safety far more approachable.
- Adopt `InlineArray` for fixed-size collections in hot paths (parsing, graphics math, audio buffers) and use `Span` instead of `UnsafeBufferPointer` for safe, zero-cost contiguous memory access.
- Integrate the `Subprocess` package for scripting and build tool use cases; the async API is far cleaner than wrapping `Process` manually.
- Add custom `Attachment.record` calls around network responses and complex data in Swift Testing to drastically speed up CI failure diagnosis.

---
_Source: WWDC25 Session 245 page (abstract, chapter summaries, code samples, and resource links)._
