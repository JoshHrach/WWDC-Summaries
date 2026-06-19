# Getting Started with Instruments
**WWDC19 · Session 411** · [Watch](https://developer.apple.com/videos/play/wwdc2019/411/)

_Platforms:_ iOS, iPadOS, macOS, watchOS, tvOS (all platforms, Xcode 11 / Instruments 11)

## Overview
This session provides a thorough introduction to Instruments for developers new to performance profiling. Starting from the Instruments UI, it walks through how to identify and fix a spinning main-thread issue using Time Profiler, then shows how adding Signposts to application code enables precise interval measurement visible in the Points of Interest instrument. The session emphasizes profiling early and often, using Release builds, and testing on real hardware.

The demo traces a macOS app (Solar System) that exhibited a Spinning Wait Cursor during a reload operation. Time Profiler exposed the root cause (data parsing happening on the main thread), and after fixing that, Signposts revealed an additional issue: the scene was being redrawn once per planet rather than once per reload batch — a 10× CPU reduction once fixed.

## Key Topics

**Instruments UI Orientation**
- Template Selector: pre-configured instrument collections (Time Profiler, Allocations, Network, File Activity, System Calls, etc.)
- Track Viewer: time-series lanes per thread, CPU core, or custom event source
- Track Pinning (new in Instruments 11): pin any track to the bottom for side-by-side comparison while scrolling
- Inspection Head + Hovering Labels: click any moment in the timeline to see per-event detail
- Detail View and Extended Detail View (Inspector): call graph, list of intervals, heaviest stack trace
- Track Filter: search tracks by name or filter by thread/CPU core
- Last Few Seconds (Windowed) Mode: reduces overhead; starts analyzing only the last few seconds of recording

**Time Profiler**
- Statistical sampling of call stacks at a fixed interval across all threads
- "Spinning" annotation in the main thread track signals a blocked main thread (Spinning Wait Cursor on Mac)
- White frames = your code; grey frames = system frameworks
- Heaviest Stack Trace in Extended Detail View immediately surfaces the hottest call path
- Option+click a disclosure triangle to auto-expand to the nearest branching point
- Click+drag in track area to filter Detail View to a specific time window

**Signposts and Points of Interest**
- `OSLog` with category `"com.apple.dt.instruments"` or the Points of Interest subsystem enables Instruments tracing
- `os_signpost(.begin, ...)` and `os_signpost(.end, ...)` mark an interval
- Swift `defer` blocks ensure `.end` is always called even on early returns
- Points of Interest track shows named intervals with duration labels
- Detail View lists all intervals, counts, averages; triple-click a track interval to select exactly that region

**Profiling Best Practices (Instruments 11)**
- Profile in Release configuration to match production compiler optimizations
- Profile on real hardware; Simulator reflects Mac resources, not device constraints
- Profile with difficult workloads and on older devices to surface real-world issues
- Use XCTest + `measure { }` with the Profile action for repeatable, testable performance regression detection
- Keep Instruments in the Dock next to Xcode as a reminder to profile throughout the development cycle

## APIs & Frameworks

**os (Unified Logging / Signposts)**
- `OSLog` — log handle for structured logging
  - `init(subsystem:category:)` — use category `"com.apple.dt.instruments"` or Points of Interest subsystem for Instruments integration
- `os_signpost(_:log:name:)` — emit a signpost event
  - `.begin` — start a named interval
  - `.end` — end a named interval
  - `.event` — a single timestamped event (no interval)
- `OSSignpostID` — unique ID to correlate begin/end pairs for concurrent intervals

**Instruments 11 (Xcode 11)** **[NEW]**
- Track Pinning **[NEW]** — pin tracks for side-by-side comparison
- Hierarchical Track Views **[NEW]** — collapsible subtree display in track area
- Time Profiler template (existing): Time Profiler + Points of Interest instruments
- Points of Interest Instrument — reads `os_signpost` events from your process

**XCTest**
- `measure { }` — runs a block multiple times and reports mean/stddev; compatible with Instruments Profile action

## Code Highlights

Adding Signposts to mark a performance interval:
```swift
import os

let log = OSLog(subsystem: "com.example.app", category: "PointsOfInterest")

func setupScene() {
    let signpostID = OSSignpostID(log: log)
    os_signpost(.begin, log: log, name: "Setup Scene", signpostID: signpostID)
    defer {
        os_signpost(.end, log: log, name: "Setup Scene", signpostID: signpostID)
    }
    // ... scene setup work ...
}
```

Moving parsing off the main thread (the root fix from the demo):
```swift
let parsingQueue = DispatchQueue(label: "com.example.parsingQueue")

func scheduleParsingTask(data: Data) {
    parsingQueue.async {
        // parse data here
        DispatchQueue.main.async {
            // update UI
        }
    }
}
```

## Takeaways
- The main thread should only handle user input and UI updates; any parsing, I/O, or computation belongs on a background queue.
- Time Profiler's "Spinning" annotation and the heaviest stack trace are the fastest path to identifying blocked-main-thread bugs.
- Add `os_signpost` intervals around operations you care about; they appear in the Points of Interest track with precise durations and counts, turning guesswork into measurement.
- Profile with `XCTest` + `measure { }` to catch performance regressions automatically before shipping.

---
_Source: WWDC19 Session 411 page (abstract, chapter summaries, code samples, and resource links)._
