# Analyze Hangs with Instruments
**WWDC23 · Session 10248** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10248/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14

## Overview
This session takes a deep dive into identifying, understanding, and fixing app hangs using the Instruments profiling tool. A hang occurs when the main thread is unresponsive long enough for the user to notice — Apple's threshold for "instant" UI response is 100ms, with hangs flagged at 250ms (micro-hang) and 500ms (full hang). The session walks through the event handling and rendering loop to explain why main-thread responsiveness is critical, then demonstrates three distinct hang patterns using the sample Backyard Birds app.

The session covers three hang archetypes: a **busy main thread hang** (main thread doing too much CPU work), an **asynchronous hang** (work dispatched to run later on the main thread blocks incoming events), and a **blocked main thread hang** (main thread waiting on a lock, semaphore, or synchronous syscall). For each, it shows which Instruments to use, how to interpret the data, and the appropriate fix.

Key guidance: use `LazyVGrid` instead of eager `Grid` to reduce redundant SwiftUI body evaluations, make async task closures `nonisolated` to avoid inheriting `@MainActor` isolation, and make synchronous properties `async` to move work off the main thread.

## Key Topics

### What is a Hang?
- 100ms threshold for "instant" feel; 250ms = micro-hang; 500ms+ = full hang.
- Hang detection measures the main thread's unresponsiveness, not just per-event latency — asynchronous work scheduled on the main thread counts too.

### Event Handling and Rendering Loop
- Hardware → OS → App main thread → Render server → Display.
- All UI updates must complete on the main thread in under 100ms.

### Busy Main Thread Hang
- Use **Time Profiler** + **SwiftUI View Body** instruments.
- Identified by high CPU usage on the main thread during the hang interval.
- Distinction: is the method running *too long*, or being called *too often* (loop)?
- Fix: replace eager `Grid` with `LazyVGrid` to avoid computing all views up front.

### Asynchronous Hangs
- Work dispatched asynchronously to the main thread (via `DispatchQueue.main.async`, `@MainActor`, `.task` modifier) blocks incoming events.
- SwiftUI's `.task` modifier inherits the `@MainActor` isolation of the `body` getter by default.
- Use **Swift Concurrency Tasks** instrument (added from Instruments library) to see which tasks run on the main thread.
- Fix: make property `get async` so the `.task` closure suspends and moves off the main actor.

### Blocked Main Thread Hang
- Identified by near-zero CPU on the main thread during the hang.
- Use **Thread State Trace** instrument to see blocked thread intervals and the exact backtrace of the blocking syscall.
- `static let shared = ...` initializers execute synchronously — accessing them inside an `@MainActor` context (even after `await`) blocks the main thread.
- A sleeping (idle) main thread does not imply a hang — look to the Hangs instrument, not Thread States, to determine actual hangs.

## APIs & Frameworks

### Instruments
- **Hangs instrument** — detects and labels intervals of main-thread unresponsiveness by severity (micro-hang, hang, severe hang)
- **Time Profiler instrument** — CPU-sampling profiler; call tree, heaviest stack trace view, source viewer
- **SwiftUI View Body instrument** — records each `body` getter execution with duration and count; orange intervals indicate slower-than-target executions
- **Swift Concurrency Tasks instrument** — tracks `Task` execution per thread; reveals tasks running on `@MainActor` unexpectedly
- **Thread State Trace instrument** — records thread blocked/running/waiting states with precise backtraces for blocking syscalls

### Swift Concurrency
- `@MainActor` — actor isolation annotation; `View.body` is implicitly `@MainActor`
- `.task` modifier — SwiftUI modifier that starts a `Task`; **inherits actor isolation of surrounding context** (i.e., `@MainActor` from `body`)
- `Task.detached` — creates a task explicitly not inheriting actor context (heavier; cancellation doesn't propagate from `.task`)
- `nonisolated` — keyword to opt a function out of actor isolation
- `get async` — makes a computed property asynchronous, allowing it to suspend and run off the main actor

### SwiftUI
- `Grid` / `GridRow` — eager layout; computes all child views on creation
- `LazyVGrid` — lazy layout; only computes views needed to fill visible area **[prefer over Grid when dealing with large datasets]**
- `LazyVGrid(columns:)` — `[GridItem]` column definition, e.g. `.adaptive(minimum:)`
- `ForEach` — used inside both `Grid` and `LazyVGrid`
- `@State` — local state for async-loaded values (e.g., `@State private var image: UIImage?`)
- `ProgressView` — placeholder while async content loads

### UIKit / Foundation
- `UIImage.preparingThumbnail(of:)` / `UIImage(byPreparingThumbnailOfSize:)` — synchronous thumbnail computation; should be moved off main thread
- `mach_msg2_trap` — syscall; appears in blocked-thread backtraces when waiting on IPC (e.g., ML model loading)
- `MLModel` — Core ML model; loading is expensive and synchronous; triggers blocking syscall if done on main thread
- `os_signpost` — manual instrumentation for measuring specific code paths in Instruments
- `NSItemProvider` — not in this session

### Xcode / Instruments UI
- Set Inspection Range (option-click) — zoom and filter to hang interval
- "Hide System Libraries" in Call Tree — filters call tree to app code only
- "Reveal in Xcode" — jumps from Instruments call tree to source
- Source Viewer — shows function implementation annotated with Time Profiler samples
- `Ctrl++` — increase track height in Instruments

## Code Highlights

Moving thumbnail loading off the main actor by making the property `async`:

```swift
// Before (synchronous — runs on @MainActor)
public var thumbnail: UIImage {
    get { /* compute and cache thumbnail */ }
}

// After (async — suspends, runs on cooperative thread pool)
public var thumbnail: UIImage {
    get async { /* compute and cache thumbnail */ }
}

// View using the async property
struct BackgroundThumbnailView: View {
    var background: BackyardBackground
    @State private var image: UIImage?

    var body: some View {
        if let image {
            Image(uiImage: image)
        } else {
            ProgressView()
                .task {
                    image = await background.thumbnail  // suspends off main actor
                }
        }
    }
}
```

## Takeaways
- All main-thread work must complete in under 100ms; use Instruments to measure before optimizing.
- Prefer `LazyVGrid` over `Grid` for large data sets — eager `Grid` computes every child view on creation, causing busy-main-thread hangs.
- SwiftUI's `.task` modifier inherits `@MainActor` from `body`; make called properties `async` to move work to the cooperative thread pool.
- Blocked main-thread hangs (low CPU, long blocked interval) require the **Thread State Trace** instrument — Time Profiler can't see idle threads.

---
_Source: WWDC23 Session 10248 page (abstract, chapter summaries, code samples, and resource links)._
