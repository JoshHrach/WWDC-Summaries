# Developing a Great Profiling Experience
**WWDC19 · Session 414** · [Watch](https://developer.apple.com/videos/play/wwdc2019/414/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
This session teaches framework and library authors how to build Custom Instruments packages that give their users professional profiling and debugging tools tied directly to their API's concepts. The session is organized around three layers: (1) adding OSSignpost trace points to production code, (2) modeling that data into meaningful Instruments tables using Eclipse-based modelers, and (3) visualizing the data through graphs, detail views, and narrative engineering types. The SolarCompression framework is used throughout as a case study, demonstrating how raw interval data becomes color-coded concurrency graphs, compression ratio aggregations, and natural-language diagnostic suggestions — all as a distributable Instruments package. Two new Instruments 11 features are introduced: hierarchical tracks and custom track scopes.

## Key Topics

### Tracing with OSSignpost
- **OSSignpost is the foundation** — low-cost tracing primitive (under 1 µs in release builds; new off-by-default dynamic categories are in the low nanosecond range). Introduced in 2018; safe to leave in production code.
- **Two kinds of signposts**: `.event` (point in time) and intervals (`.begin` / `.end` pairs matching on `OSSignpostID`). Both implicitly record a high-accuracy timestamp; intervals support overlap across threads via signpost IDs.
- **Three recording modes**: default (ring buffer, lowest cost), streaming (immediate, ~10× higher cost), and the two new dynamic tracing categories (off by default, only enabled when a Custom Instrument is recording). **[NEW dynamic categories]**
- **OSLog handle** is a named namespace (subsystem + category). Static strings (subsystem, category, signpost name, format string) are stored as offsets into the binary — low overhead; don't change them without updating the package.
- **Four tracing best practices:**
  1. Always close intervals — use Swift `defer` to guarantee `.end` even on early returns and thrown errors.
  2. Avoid logging duplicate data in both begin and end; log it once at first availability.
  3. Guard expensive data computation behind `OSSignposter.signpostsEnabled` (only true when Instruments is recording with the dynamic category).
  4. Trace only useful data — move signpost calls after guard/precondition checks for short intervals that would clutter data.
- **Stable trace points** — OSSignpost is designed to tolerate compiler inlining; only the static strings matter for Custom Instrument compatibility.
- **Throughput estimate**: at ~0.5 µs per signpost and a 1% CPU budget on a single core, ~20,000 signposts/second is achievable — enough for 83 intervals per frame at 120 FPS on iPad Pro.

### Modeling in Instruments
- **Architecture** — Instruments stores everything in tables. Schemas define table structure (point schema: timestamp column; interval schema: timestamp + duration columns). Modelers observe input tables, apply domain logic, and emit output tables.
- **Auto-generated modelers** — An `os-signpost` interval schema in XML auto-generates a modeler from OSSignpost data without Eclipse code.
- **Custom Eclipse modelers** — needed when combining multiple log handles or built-in data sources, maintaining state, tracking event order, or computing derived metrics (e.g., quantized utilization averages).
- **Start with the built-in OSSignpost tool** — verify that interval data and metadata look correct before building a custom schema.

### Visualization
- **Goal: tell a story, not just show data** — raw intervals convey meaning only to the author; the Custom Instrument should teach users to diagnose problems without help.
- **Graphs draw eyes** — top-level tracks should highlight problem areas (color coding, severity columns).
- **Event (point) visualization patterns:**
  - Histogram — equal-importance events; bar height = density at a glance; `best-for-resolution` element switches between histogram (zoomed out) and individual events (zoomed in).
  - Dedicated lane — for critical events that must stand out.
- **Interval visualization patterns:**
  - Qualified/instance plots — for bounded overlapping intervals.
  - Hierarchical tracks — **new in Instruments 11** — nested tracks for large numbers of concurrent intervals; filterable and pinnable. **[NEW]**
  - Quantized load average — for counting active intervals over a time bucket; color extremes for severity.
  - Interval count per time bucket — highlights high-frequency-short-interval inefficiency.
  - Overlap-degree tracks — shows exact duration of N concurrent intervals.
- **Narrative engineering type** — plain-language explanations in detail views for contextual, actionable guidance. Example: "File size decreased by 1% — compression may not be necessary for .zip archives; consider LZMA for better ratio."
- **Aggregation detail views** — tabular summaries with Min/Max/Average/StdDev per column; column ordering and naming communicates what is important.
- **Custom track scopes** — **new in Instruments 11** — named filter views that show only specific tracks/instruments; saveable in templates; for sharing focused debugging perspectives with a team. **[NEW]**

## APIs & Frameworks

### OSLog / OSSignpost
- `OSLog(subsystem:category:)` — creates named log handle
- `OSSignposter` (Swift unified API) — `.begin(name:id:)`, `.end(name:id:)`, `.event(name:id:)` **[NEW Swift API]**
- `os_signpost(.begin, log:, name:, signpostID:, format:, ...)` — C macro form
- `os_signpost(.end, log:, name:, signpostID:)` — close interval
- `OSSignpostID(log:object:)` — unique per-interval identifier for matching begin/end across threads
- `OSSignposter.signpostsEnabled` — Bool; gated by whether a custom instrument's dynamic category is currently recording **[NEW]**
- Dynamic tracing categories (new): off-by-default log categories that activate only under Custom Instrument recording **[NEW]**

### Instruments Custom Instruments **[NEW XML schema elements]**
- `os-signpost` schema — auto-generates modeler from OSSignpost data
- `point-schema` — custom output table with timestamp column
- `interval-schema` — custom output table with timestamp + duration columns
- `modeler` — Eclipse-based modeler consuming input tables, emitting output
- `hierarchy` — nested track grouping **[NEW in Instruments 11]**
- `track-scope` — named filter/branch set for perspectives **[NEW in Instruments 11]**
- `narrative` engineering type — column type for plain-text diagnostic messages
- `histogram` graphing element — density visualization for point data; `best-for-resolution` sub-element
- Qualified / instance plots — vertical lane splitting for bounded overlapping intervals

## Code Highlights

OSSignpost interval with defer for guaranteed close:

```swift
import OSLog

let log = OSLog(subsystem: "com.example.SolarCompression",
                category: "CompressionManager")

func compress(item: CompressionItem) throws {
    let id = OSSignpostID(log: log, object: item)
    os_signpost(.begin, log: log, name: "CompressionExecution",
                signpostID: id,
                "algorithm=%{public}@, kind=%{public}@, src=%{public}@",
                item.algorithm, item.kind, item.sourcePath)
    defer {
        os_signpost(.end, log: log, name: "CompressionExecution",
                    signpostID: id,
                    "destSize=%ld", item.destinationSize)
    }
    try startCompression(item: item) // may throw — defer ensures .end fires
}
```

Gating expensive work behind signpost-enabled check:

```swift
let signposter = OSSignposter(subsystem: "com.example.framework",
                              category: .dynamicTracing)
if signposter.signpostsEnabled {
    let ratio = computeExpensiveCompressionRatio(item)
    signposter.emitEvent("CompressionRatio", "\(ratio)")
}
```

## Takeaways

- Custom Instruments are the best way to scale debugging expertise from one expert to an entire community — they teach, diagnose, and build trust in a framework more effectively than documentation alone.
- Always use Swift `defer` to close OSSignpost intervals; an unclosed interval permanently degrades Instruments' analysis performance.
- Use the new off-by-default dynamic tracing categories for high-volume or expensive trace points: they incur zero cost when no Custom Instrument is recording and never pollute the built-in OSSignpost tracks.
- Visualization design matters as much as data correctness — top-level graphs should draw the user's eye to problem areas immediately (color severity, load averages, histogram spikes), with detail views providing actionable narrative explanations rather than raw numbers.

---
_Source: WWDC19 Session 414 page (abstract, full transcript, and resource links)._
