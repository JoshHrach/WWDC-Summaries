# Meet the New MetricKit

**WWDC26 · Session 222** · [Watch](https://developer.apple.com/videos/play/wwdc2026/222/)

_Platforms:_ iOS 27 · macOS · iPadOS · tvOS · watchOS

## Overview

MetricKit has been rebuilt from the ground up in iOS 27 with a new Swift-first, `async/await`-based API that replaces the older delegate callback model. The framework continues its mission of delivering real-world performance data collected from user devices, but now with a cleaner API, new metric categories (Metal frame rate, storage), new diagnostic types (memory exceptions), and — most significantly — a new integration with the `StateReporting` framework that allows performance data to be segmented by app state.

The three-chapter structure of the session mirrors MetricKit's three pillars: Metrics (daily aggregated performance reports), Diagnostics (real-time issue reports on crashes and hangs), and Context (state-aware segmentation via StateReporting). Each pillar now uses an `AsyncSequence` pattern where the developer iterates over `metricReports` or `diagnosticReports` from a `MetricManager` instance.

The StateReporting integration is the headline new feature: by registering named domains and reporting state transitions (e.g., "user navigated to Reports tab"), MetricKit can aggregate performance data separately per state rather than blending all activity together. The `@ReportableMetadata` macro enables attaching structured typed metadata to state transitions, providing even more granular context for server-side analysis.

## Key Topics

### Metrics
- `MetricManager` delivers daily reports as `MetricReport` values via an `AsyncSequence`.
- Reports contain `IntervalEntry` values organized by metric groups.
- **[NEW]** Metric groups: CPU, memory, launch time, hangs, disk I/O, display (Metal frame rate), storage.
- Reports are `Codable` — encode as JSON and ship to your analytics server.
- Access specific metric values by filtering `entry.values` on `.metricGroup`.
- `fullDayEntry` provides a single aggregate covering the entire reporting window.

### Diagnostics
- `MetricManager` also delivers `DiagnosticReport` values immediately when an issue occurs (no 24-hour delay).
- `DiagnosticReport.result` is an enum with cases: `.crash`, `.hang`, `.diskWriteException`, `.cpuException`, and **[NEW]** `.memoryException`.
- Crash diagnostics include `callStackTree`, `terminationReason`, and **[NEW]** `terminationCategory`.
- Reports are `Codable` for server upload.

### Context (StateReporting)
- **[NEW]** `StateReporting` framework — report named states and transitions throughout the app lifecycle.
- `StateReportingDomain` — a named domain representing a logical area of the app (e.g., a specific tab flow).
- `StateReporter` — reports transitions to named state strings.
- `MetricManager(enabledStateReportingDomains:)` — initialize with domains to enable per-state aggregation.
- **[NEW]** `@ReportableMetadata` macro — attach structured Swift types as metadata to state transitions.
- `MetricReport.EncodingFormat.byStateReportingDomain` — encode reports segmented by state domain for server analysis.
- `StateEntry` — report entry values scoped to a specific state.

## APIs & Frameworks

**MetricKit (Rebuilt Swift-first API)**
- `MetricManager` — **[NEW/REVISED]** main entry point; now uses `async/await`
- `MetricManager.metricReports` — `AsyncSequence` of `MetricReport` **[NEW]**
- `MetricManager.diagnosticReports` — `AsyncSequence` of `DiagnosticReport` **[NEW]**
- `MetricManager(enabledStateReportingDomains:)` — init with StateReporting domains **[NEW]**
- `MetricReport` — daily aggregated performance report; `Codable`
- `MetricReport.intervalEntries` — collection of `IntervalEntry` values
- `MetricReport.intervalEntries.fullDayEntry` — single full-day aggregate `IntervalEntry`
- `IntervalEntry` — time-windowed metric collection
- `IntervalEntry.values` — array of metric values filterable by `.metricGroup`
- Metric groups: `.memory`, `.cpu`, `.display`, `.launch`, `.hangs`, `.diskIO`, `.storage` **[NEW: display, storage]**
- `.peakMemory(Measurement<UnitInformationStorage>)` — peak memory value
- `DiagnosticReport` — real-time issue diagnostic; `Codable`
- `DiagnosticReport.result` — enum: `.crash`, `.hang`, `.diskWriteException`, `.cpuException`, `.memoryException` **[NEW: .memoryException]**
- `CrashDiagnostic.callStackTree` — symbolicated backtrace
- `CrashDiagnostic.terminationReason` — reason string
- `CrashDiagnostic.terminationCategory` — **[NEW]** termination category enum
- `HangDiagnostic` — hang duration and call stack
- `MetricReport.EncodingFormatKey` — `JSONEncoder.userInfo` key for encoding format **[NEW]**
- `MetricReport.EncodingFormat.byStateReportingDomain` — per-state segmented encoding **[NEW]**
- `StateEntry` — metric entry scoped to a named state **[NEW]**

**StateReporting Framework (NEW)**
- `StateReportingDomain` — named domain identifier (e.g., `"com.myapp.tabs"`) **[NEW]**
- `StateReporter` — reports transitions; retrieve with `StateReporter.reporter(for:)` or `reporter(for:stableMetadata:)` **[NEW]**
- `StateReporter.reportTransition(to:)` — report a state transition **[NEW]**
- `StateReporter.reportTransition(to:stableMetadata:)` — report with structured metadata **[NEW]**
- `@ReportableMetadata` — macro to mark a struct as state metadata **[NEW]**

**Related Documentation**
- [MetricKit](https://developer.apple.com/documentation/MetricKit)
- [Getting started with StateReporting](https://developer.apple.com/documentation/StateReporting/getting-started-with-statereporting)
- [Analyzing app performance with MetricKit](https://developer.apple.com/documentation/MetricKit/analyzing-app-performance-with-metrickit)
- [Track performance by app state using MetricKit](https://developer.apple.com/documentation/MetricKit/track-performance-by-app-state-using-metrickit)

## Code Highlights

Receiving metrics with the new async API:

```swift
import MetricKit

let manager = MetricManager()

for await report in manager.metricReports {
    let fullDayEntry = report.intervalEntries.fullDayEntry
    let memoryMetrics = fullDayEntry.values.filter { $0.metricGroup == .memory }
    for metric in memoryMetrics {
        switch metric {
        case .peakMemory(let peak): processPeakMemory(peak)
        default: break
        }
    }
}
```

Receiving diagnostics:

```swift
for await report in manager.diagnosticReports {
    switch report.result {
    case .crash(let crash):
        processCrash(backtrace: crash.callStackTree,
                     reason: crash.terminationReason,
                     category: crash.terminationCategory)
    case .hang(let hang): processHangDiagnostic(hang)
    default: break
    }
}
```

StateReporting integration with structured metadata:

```swift
import StateReporting

@ReportableMetadata
struct ViewConfiguration {
    let listSize: String
    let isSorted: Bool
}

let domain = StateReportingDomain("com.metrickitsample.tabs")
let manager = MetricManager(enabledStateReportingDomains: [domain])
let reporter = StateReporter.reporter(for: domain.rawValue,
                                       stableMetadata: ViewConfiguration.self)
reporter.reportTransition(to: "Reports",
    stableMetadata: ViewConfiguration(listSize: "large", isSorted: false))
```

## Takeaways

- MetricKit's new Swift-first `async/await` API removes the delegate boilerplate and makes it practical to add performance monitoring to any app with a few lines of code.
- The new `StateReporting` integration is the most impactful new feature: blended metrics across all app states hide the true cause of regressions; per-state data pinpoints them.
- `@ReportableMetadata` lets teams attach typed contextual metadata (feature flags, configuration values) to state transitions, enabling much richer server-side analysis than string labels alone.
- MetricKit diagnostic reports arrive in real time (no 24-hour delay), making them complementary to — not a replacement for — crash reporting services.

---
_Source: WWDC26 Session 222 page (abstract, chapter summaries, code samples, and resource links)._
