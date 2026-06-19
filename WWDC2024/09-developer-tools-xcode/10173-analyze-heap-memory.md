# Analyze Heap Memory
**WWDC24 · Session 10173** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10173/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, watchOS, visionOS (Instruments/Xcode tools run on macOS)

## Overview
This session provides a deep dive into heap memory—where an app's reference types and dynamic allocations live—and how to measure, diagnose, and fix common heap problems using Instruments and Xcode. The heap is the dominant contributor to an app's dirty memory footprint, which counts against process memory limits and can cause crashes or background task terminations.

Three categories of heap problems are addressed through live demos using the DestinationVideo sample app: transient memory growth (spikes caused by autoreleased objects accumulating in loops), persistent memory growth (objects that should be freed but aren't, typically due to incorrect cache keys), and memory leaks (reference cycles preventing deallocation). A final section covers runtime performance—the overhead of `weak` vs. `unowned` references and ARC retain/release traffic.

## Key Topics
- **Heap memory overview** — virtual memory regions, clean/dirty/swapped pages, malloc internals, 16-byte minimum alignment
- **Tools overview** — Xcode Memory Report, Instruments Allocations template, Instruments Leaks template, Memory Graph Debugger, CLI tools (`heap`, `vmmap`, `leaks`, `malloc_history`)
- **Transient growth** — memory spikes from autorelease pool accumulation in loops; fix with nested `autoreleasepool { }` scope per iteration
- **Persistent growth** — allocations that never free; detected via Allocations "Mark Generation"; root cause found via Memory Graph Debugger
- **Memory leaks** — unreachable allocations from reference cycles; closure contexts strongly capturing their owners; fixed with `[weak self]` or `[unowned self]` capture lists
- **`weak` vs. `unowned`** — performance and memory tradeoffs; when to use each
- **ARC overhead reduction** — whole-module optimization, `objc_direct`, `objc_externally_retained`, minimizing reference types in hot-path structs

## APIs & Frameworks
### Instruments
- **Allocations instrument** — records all malloc/free events; call trees, "Created & Still Living", "Created & Destroyed" lifespan filters; `Mark Generation` feature for persistent growth isolation
- **Leaks instrument** — periodic snapshots; detects unreachable allocations
- **VM Tracker** — snapshots virtual memory regions (dirty/swapped pages)
- `MallocStackLogging` — records backtraces for every allocation; enable via Xcode scheme diagnostics; required for backtrace-enriched Memory Graphs

### Xcode
- **Memory Graph Debugger** — captures a snapshot of all heap allocations and references; shows strong/weak/unowned/conservative reference edges; links to source via stack traces; `Show only leaked allocations` filter
- `Reflection Metadata Level` build setting — set to `All` for best Swift reference scanning accuracy in Memory Graph Debugger

### Swift / ARC
- `autoreleasepool { }` — drain Objective-C autoreleased objects per loop iteration to prevent transient spikes
- `weak var` — optional, nil-able non-retaining reference; allocates a `SwiftWeakReference` side allocation
- `unowned let` — non-optional, non-retaining; no side allocation; crashes if destination deallocates while reference exists
- `[weak self]` / `[unowned self]` — capture list syntax to break closure reference cycles
- Swift 6 / `Sendable` — compile-time data-race detection (related: safe shared state avoids whole categories of heap bugs)
- `-whole-module-optimization` — enables more ARC elision via inlining

### Objective-C / C
- `objc_direct` — marks ObjC methods as direct calls (no objc_msgSend), enabling inlining and reducing retain/release traffic
- `objc_externally_retained` — tells compiler parameter lifetime is guaranteed externally; eliminates unnecessary retain/release at call sites
- `UnsafeMutablePointer.allocate(capacity:)` / `.deallocate()` — manual C-level allocation; leaked if deallocate is omitted

## Code Highlights
```swift
// Fix transient growth: nested autorelease pool per loop iteration
func loadThumbnails(with renderer: ThumbnailRenderer) {
    for photoURL in urls {
        autoreleasepool {
            renderer.faultThumbnail(from: photoURL)
        }
    }
}

// Fix leak: weak capture in completion closure
let loader = ThumbnailLoader(bundle: .main, completionQueue: .main)
loader.completionHandler = { [weak renderer] in
    guard let renderer else { return }
    self.thumbnails = renderer.images
}

// Fix implicit self capture in method-as-closure (causes cycle)
// Bad:
generator = defaultAction          // implicitly captures self strongly

// Good (unowned — closure has same lifetime as self):
generator = { [unowned self] data in self.defaultAction(data) }

// Fix "noreturn" function leak: store in global
static var singleton: Server?
func beginServer() {
    Self.singleton = Server(delegate: self)
    dispatchMain()
}
```

## Takeaways
- Enable `MallocStackLogging` in your scheme's diagnostics before profiling—backtraces in the Memory Graph Debugger are essential for pinpointing leak sources
- Autorelease pool accumulation inside loops is a common, easy-to-miss cause of memory spikes in Swift code that calls Objective-C APIs; wrap loop bodies in `autoreleasepool { }`
- Use `unowned` instead of `weak` only when you can guarantee the reference won't outlive its destination; otherwise `weak` is the safe default (at the cost of one extra heap allocation per weakly-referenced object)
- The Memory Graph Debugger's "Show only leaked allocations" filter plus source-linked stack traces makes finding and fixing reference cycles a fast, repeatable workflow

---
_Source: WWDC24 Session 10173 page (abstract, chapter summaries, code samples, and resource links)._
