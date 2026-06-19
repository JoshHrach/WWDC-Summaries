# Ultimate Application Performance Survival Guide
**WWDC21 · Session 10181** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10181/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session is a comprehensive tour of Apple's app performance toolset, covering eight key metrics and the tools used to measure, diagnose, and fix regressions in each area. The session is structured as a practical guide covering Battery Usage, Hang Rate, Scrolling, Disk Writes, Launch Time, Terminations, and Memory — with specific tooling recommendations at each stage of the development lifecycle (development, pre-release testing, post-release monitoring).

The five main tools covered are: Xcode Organizer (aggregate field data, new Regressions pane in Xcode 13, new Terminations pane), MetricKit (on-device telemetry, now with instant diagnostic delivery in iOS 15), Instruments (Xcode profiling templates), XCTest performance tests, and the App Store Connect API (programmatic access to all Organizer data).

New in iOS 15/macOS 12: MetricKit diagnostics (hangs, crashes) now delivered immediately on next app launch rather than once daily; `mxSignpostAnimationIntervalBegin` for custom animation hitch-rate telemetry.

## Key Topics

### Eight Key Performance Metrics
1. **Battery Usage** — CPU, networking, location subsystems; track with Energy Gauge in Xcode Debug Navigator, Energy Organizer
2. **Hang Rate** — unresponsive for ≥250ms; new instant MetricKit delivery in iOS 15 **[NEW]**
3. **Scrolling** — hitch rate; `XCTOSSignpostMetric.scrollDecelerationMetric`, `mxSignpostAnimationIntervalBegin` **[NEW]**
4. **Disk Writes** — NAND wear, slow I/O; File Activity Instruments template, `XCTStorageMetric`, Disk Writes Organizer Insights **[NEW]**
5. **Launch Time** — time to first frame; App Launch Instruments template, `XCTApplicationLaunchMetric`, Launch Organizer
6. **Terminations** — memory limit, launch timeout, background kills; new Terminations Organizer pane **[NEW]**
7. **Memory** — peak and suspended memory; Leaks/Allocations/VM Tracker Instruments, MetricKit, `mxSignpost` intervals
8. **MXSignposts** — custom telemetry bookmarks around critical code sections

### Xcode Organizer Improvements (Xcode 13)
- New **Regressions pane** **[NEW]**: isolates metrics that increased significantly in the most recent version
- New **Terminations pane** **[NEW]**: trend data for process terminations categorized by reason
- Disk Writes Reports now include **Insights** tab with optimization suggestions **[NEW]**
- All Organizer data also accessible via App Store Connect API

### MetricKit Improvements (iOS 15 / macOS 12)
- Hang diagnostics, crash diagnostics now delivered **immediately on next app launch** (previously 24-hour cadence) **[NEW]**
- `mxSignpostAnimationIntervalBegin(log:name:)` **[NEW]**: marks the start of a custom animation interval; hitch-rate telemetry collected automatically during interval
- `mxSignpost(.end, log:name:)`: marks end of interval, delivers collected hitch-rate data
- MetricKit on macOS **[NEW]** (see also session 10203)

### XCTest Performance Testing
- `XCTOSSignpostMetric.scrollDecelerationMetric`: measure scroll performance
- `XCTStorageMetric(application:)`: measure disk writes
- `XCTApplicationLaunchMetric`: measure launch time
- `XCTMeasureOptions.invocationOptions = [.manuallyStop]` or `[.manuallyStart]`: control when measurement begins/ends within a measure block
- `stopMeasuring()` / `startMeasuring()`: explicit measurement start/stop

### Instruments Templates
- **Energy Gauge** (Xcode Debug Navigator): CPU utilization, wake overhead during development
- **Time Profiler**: call stacks, thermal state, CPU usage
- **Thread State Trace**: thread blocking, OS scheduling timeline
- **System Call Trace**: system call narrative and duration
- **App Launch**: 5-second time profile of app launch
- **File Activity**: file system operations and system calls
- **Leaks**: heap leak detection
- **Allocations**: memory lifecycle analysis
- **VM Tracker**: virtual memory usage over time
- **Location Energy Model**: Core Location power impact

## APIs & Frameworks

- `MetricKit` framework
- `MXMetricManager`
- `MXMetricManager.shared`
- `MXMetricManager.add(_:)` / `remove(_:)`
- `MXMetricManager.makeLogHandle(category:)`
- `MXMetricManagerSubscriber` protocol
- `MXMetricManagerSubscriber.didReceive(_:)` (MXMetricPayload) — daily metrics
- `MXMetricManagerSubscriber.didReceive(_:)` (MXDiagnosticPayload) — diagnostics (now immediate in iOS 15)
- `MXMetricPayload`
- `MXDiagnosticPayload`
- `MXDiagnosticPayload.crashDiagnostics`
- `mxSignpost(_:log:name:)` — custom telemetry signpost
- `mxSignpostAnimationIntervalBegin(log:name:)` **[NEW]** — animation interval begin with hitch-rate collection
- `XCTest` framework
- `XCTOSSignpostMetric.scrollDecelerationMetric`
- `XCTStorageMetric(application:)`
- `XCTApplicationLaunchMetric`
- `XCTMeasureOptions`
- `XCTMeasureOptions.invocationOptions`
- `measure(metrics:options:_:)`
- `startMeasuring()` / `stopMeasuring()`
- App Store Connect API (programmatic access to Organizer metrics data)

## Code Highlights

MetricKit subscriber setup:
```swift
class AppMetrics: MXMetricManagerSubscriber {
    init() { MXMetricManager.shared.add(self) }
    deinit { MXMetricManager.shared.remove(self) }
    func didReceive(_ payloads: [MXMetricPayload]) { /* process daily metrics */ }
    func didReceive(_ payloads: [MXDiagnosticPayload]) { /* process instant diagnostics */ }
}
```

Custom animation hitch-rate telemetry (iOS 15):
```swift
func startAnimating() {
    mxSignpostAnimationIntervalBegin(
        log: MXMetricManager.makeLogHandle(category: "animation_telemetry"),
        name: "custom_animation")
}
func animationDidComplete() {
    mxSignpost(.end,
        log: MXMetricManager.makeLogHandle(category: "animation_telemetry"),
        name: "custom_animation")
}
```

Disk write XCTest with manual start:
```swift
measure(metrics: [XCTStorageMetric(application: app)], options: options) {
    app.launch()
    startMeasuring()
    app.cells.firstMatch.buttons["Save meal"].tap()
}
```

## Takeaways

- Use the new Regressions pane in Xcode Organizer 13 to immediately identify which metrics regressed in the latest app version; no more searching through eight separate metric views.
- MetricKit diagnostics (hangs, crashes) are now delivered on the very next app launch in iOS 15 — integrate `MXMetricManagerSubscriber` and process `MXDiagnosticPayload` for near-real-time field diagnostics.
- Write XCTest performance tests for scroll (`scrollDecelerationMetric`), disk (`XCTStorageMetric`), and launch (`XCTApplicationLaunchMetric`) metrics to catch regressions before they ship.
- Use `mxSignpostAnimationIntervalBegin` to collect hitch-rate telemetry on custom animations in production, providing data that standard crash/hang reports would miss.

---
_Source: WWDC21 Session 10181 page (abstract, chapter summaries, code samples, and resource links)._
