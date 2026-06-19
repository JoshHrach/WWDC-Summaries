# Measuring Performance Using Logging
**WWDC18 · Session 405** · [Watch](https://developer.apple.com/videos/play/wwdc2018/405/)

_Platforms:_ iOS 12, macOS Mojave 10.14, watchOS 5, tvOS 12

## Overview
Signposts are a new member of the `os_log` family introduced in 2018, designed specifically for performance measurement. Unlike general-purpose logging, signposts are optimized to mark the beginning and end of time intervals within your code and integrate directly with Instruments for visualization and analysis.

The session demonstrates how to annotate asynchronous and concurrent operations using signpost IDs so overlapping intervals can be tracked distinctly. Metadata can be attached to signpost calls using OSLog format strings, enabling rich statistical analysis of performance-relevant values directly in Instruments.

Instruments 10 introduces three complementary features that work with signpost data: the new os_signpost instrument, Points of Interest, and Custom Instruments — together providing a complete picture of your app's runtime behavior and performance characteristics.

## Key Topics

### Signpost Basics
- Using `os_signpost(.begin)` and `os_signpost(.end)` to bracket intervals of work
- Associating begin/end pairs using a shared signpost name (string literal)
- Creating a log handle with subsystem and category identifiers

### Concurrent Intervals and Signpost IDs
- Using `OSSignpostID` to distinguish overlapping intervals of the same named operation
- Creating signpost IDs from a log handle or from an object instance
- Passing the same signpost ID at both the begin and end call sites

### Metadata Attachment
- Adding OSLog format strings to signpost calls to pass numeric and string values
- Using engineering type annotations (e.g., `Xcode:size-in-bytes`) to hint Instruments on how to interpret and display values
- Emitting single-point-in-time signposts using `os_signpost(.event)`

### Enabling and Disabling Signposts
- Using `OSLog.disabled` to turn signposts into near-no-ops at runtime without changing call sites
- Gating expensive instrumentation-only work with the `signpostsEnabled` property

### Instruments 10 Features
- **os_signpost instrument**: records, visualizes, and aggregates all signpost intervals and events
- **Points of Interest**: promoting key user-facing events to a special system category visible in a dedicated Instruments track
- **Custom Instruments**: authoring an XML-based `.instrpkg` to reshape signpost data into tailored visualizations and summary tables
- Switching between Immediate Mode and Windowed (Last N Seconds) recording modes to minimize overhead during high-frequency signpost emission

## APIs & Frameworks

- **`os.signpost` module** **[NEW]**
- **`OSLog`** — log handle creation with `OSLog(subsystem:category:)` and `OSLog.disabled`
- **`OSSignpostID`** **[NEW]** — `OSSignpostID(log:)`, `OSSignpostID(log:object:)`
- **`os_signpost(_:log:name:signpostID:_:)`** **[NEW]** — function with `.begin`, `.end`, `.event` types
- **`OSSignpostType`** **[NEW]** — `.begin`, `.end`, `.event`
- **`OSLog.signpostsEnabled`** **[NEW]** — property to check if signposts are active for a handle
- **`OSLog.Category.pointsOfInterest`** **[NEW]** — special system category for Points of Interest instrument
- **Instruments 10** — os_signpost instrument, Points of Interest instrument track
- **Custom Instruments / `.instrpkg`** **[NEW]** — XML-based package for defining custom instrument visualizations, detail views, summary tables, and timeslice views

## Code Highlights

Creating a log handle and emitting a basic interval:
```swift
import os.signpost

let log = OSLog(subsystem: "com.example.trailblazer", category: "networking")

os_signpost(.begin, log: log, name: "Background Image")
fetchAsset(name: name)
os_signpost(.end, log: log, name: "Background Image")
```

Using a signpost ID for concurrent intervals:
```swift
let spid = OSSignpostID(log: log, object: downloader)
os_signpost(.begin, log: log, name: "Background Image", signpostID: spid,
            "Downloading %{public}s", imageName)
// ... async work ...
os_signpost(.end, log: log, name: "Background Image", signpostID: spid,
            "Finished with size %{Xcode:size-in-bytes}d", byteCount)
```

Emitting a Point of Interest event:
```swift
let poi = OSLog(subsystem: "com.example.trailblazer", category: .pointsOfInterest)
os_signpost(.event, log: poi, name: "Detail Appeared", "Trail: %{public}s", trailName)
```

Conditionally disabling signposts:
```swift
let log: OSLog = ProcessInfo.processInfo.environment["SIGNPOSTS_ENABLED"] != nil
    ? OSLog(subsystem: "com.example.app", category: "perf")
    : .disabled
```

## Takeaways
- Signposts add minimal runtime overhead and are designed to be left in production code paths; use `OSLog.disabled` to gate expensive instrumentation at runtime.
- Signpost IDs are essential for correctly associating begin/end pairs in concurrent and asynchronous code.
- The Points of Interest category surfaces key user-facing events in every Instruments trace without requiring the full os_signpost instrument.
- Custom Instruments packages (XML, ~115 lines) can reshape raw signpost data into domain-specific visualizations meaningful to the whole team.

---
_Source: WWDC18 Session 405 page (abstract, chapter summaries, code samples, and resource links)._
