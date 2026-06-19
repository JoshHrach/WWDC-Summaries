# Improving Battery Life and Performance
**WWDC19 · Session 417** · [Watch](https://developer.apple.com/videos/play/wwdc2019/417/)

_Platforms:_ iOS 13, iPadOS 13 (Xcode 11 + MetricKit; Xcode Metrics Organizer)

## Overview
iOS 13 and Xcode 11 introduce three new tools that bring quantitative battery and performance metrics into every phase of the development lifecycle. Previously, developers could use Instruments, Debug Gauges, and Xcode Organizer crash/energy logs, but lacked a structured way to measure, compare, and regress-test numeric metrics. The three new tools — XCTest Metrics, MetricKit, and Xcode Metrics Organizer — cover development/test, beta, and production release stages respectively.

The session organizes performance metrics into two families: battery metrics (processing/CPU/GPU, location, display average pixel luminance, networking, Bluetooth, multimedia, camera) and performance metrics (hangs, disk logical writes, application launch/resume times, peak and suspended memory, custom intervals). All three tools report subsets of these same metrics, allowing developers to track regressions from first commit through wide release.

## Key Topics

**XCTest Metrics (Development & Testing)**
- Extend existing `measure { }` blocks by passing metric objects: `XCTCPUMetric`, `XCTMemoryMetric`, `XCTStorageMetric`, `XCTClockMetric`, `XCTOSSignpostMetric`
- New UI test targets automatically include a free application launch test — no code required
- Set baselines (average + standard deviation) on each metric; tests fail automatically on regression
- Useful for A/B comparison of algorithm variants without a full Instruments trace
- Disable sanitizers and detach the debugger before running performance tests to minimize overhead; Test Plans make it easy to create a performance-only scheme configuration

**MetricKit (Beta and Field — iOS 13)**
- On-device framework for battery and performance metric collection from real users in the field
- Conforms to `MXMetricManagerSubscriber`; receives `MXMetricPayload` at most once per 24-hour period with aggregated metrics
- Payload includes: launch/resume histograms, hang duration histograms, foreground/background CPU and GPU time, location accuracy usage, network upload/download bytes, disk I/O, memory, display average pixel luminance
- `mxSignpost` API wraps `os_signpost` to bookmark critical code sections; MetricKit aggregates per-region CPU time and count in `MXSignpostMetric`
- Use `MXMetricManager.makeLogHandle(category:)` to get a log handle, then call `mxSignpost(.begin/.end, log:name:)` around critical sections
- Test integration: Xcode 11 adds "Simulate MetricKit Payloads" in the Debug menu to trigger a dummy payload delivery without waiting 24 hours

**Xcode Metrics Organizer (Production — No Code Changes)**
- New "Metrics" tab in Window > Organizer in Xcode 11
- Shows aggregated battery and performance data for all App Store versions with no code changes to the app
- Metrics: battery life (onscreen and background, subdivided by processing/networking/display/location), launch time, hang rate (seconds/hour), peak memory, average suspended memory, disk writes
- Compare across app versions (x-axis = version, y-axis = metric value)
- Segment by device family (all iPhone, all iPad, specific model) and by user population percentile (50th/90th)
- Drill from Metrics to Energy Exception Reports (energy tab) for stack-frame analysis
- Insights appear only when sufficient usage data meets Apple's privacy/threshold requirements

**Key Metrics Explained**
- Average Pixel Luminance (APL): on OLED devices (iPhone X/XS), lighter UI colors consume more energy; darker UI (Dark Mode) reduces APL and display battery drain
- Hang rate: time (seconds/hour) the app is unresponsive; target is 0; fix by moving work off main thread via GCD async dispatch
- Disk logical writes: measure to detect unexpected writes; apply write-coalescing strategies
- Location accuracy: use the lowest accuracy bucket sufficient for the use case; leaving location running when not needed is a common battery drain source

## APIs & Frameworks

**XCTest** (Xcode 11) **[NEW metrics]**
- `XCTCPUMetric` **[NEW]** — measures CPU time in `measure` blocks
- `XCTMemoryMetric` **[NEW]** — measures memory usage in `measure` blocks
- `XCTStorageMetric` **[NEW]** — measures disk writes in `measure` blocks
- `XCTClockMetric` **[NEW]** — measures wall-clock elapsed time (replaces implicit timing)
- `XCTOSSignpostMetric` **[NEW]** — measures `os_signpost`-delimited intervals in `measure` blocks
- `XCTestCase.measure(metrics:block:)` **[NEW]** — overload accepting `[XCTMetric]`
- `XCTestCase.measure(block:)` — existing API; now uses `XCTClockMetric` internally
- Baseline support: `setMeasurementBaseline(_:forMetric:)` / `resetMeasurementBaseline(forMetric:)`

**MetricKit** (iOS 13) **[NEW framework]**
- `MXMetricManager` **[NEW]** — singleton; `shared`; `add(_:)` to subscribe
- `MXMetricManagerSubscriber` **[NEW]** — protocol; implement `didReceive(_:)` for payload delivery
- `MXMetricPayload` **[NEW]** — 24-hour aggregate payload containing all metric categories
- `MXCPUMetric` **[NEW]** — `cumulativeCPUTime`
- `MXGPUMetric` **[NEW]** — `cumulativeGPUTime`
- `MXMemoryMetric` **[NEW]** — `peakMemoryUsage`, `averageSuspendedMemory`
- `MXDiskIOMetric` **[NEW]** — `cumulativeLogicalWrites`
- `MXDisplayMetric` **[NEW]** — `averagePixelLuminance`
- `MXNetworkTransferMetric` **[NEW]** — `cumulativeCellularUpload`, `cumulativeCellularDownload`, `cumulativeWifiUpload`, `cumulativeWifiDownload`
- `MXLocationActivityMetric` **[NEW]** — cumulative seconds per accuracy tier (best, navigation, reduced, etc.)
- `MXAppLaunchMetric` **[NEW]** — `histogrammedTimeToFirstDraw`, `histogrammedApplicationResumeTime`
- `MXAppResponsivenessMetric` **[NEW]** — `histogrammedApplicationHangTime`
- `MXSignpostMetric` **[NEW]** — per-region CPU time and invocation count from `mxSignpost`
- `MXMetricManager.makeLogHandle(category:)` **[NEW]** — creates `OSLog` handle for mxSignpost
- `mxSignpost(_:log:name:)` **[NEW]** — wraps `os_signpost`; signals MetricKit to collect per-region metrics

**Xcode Metrics Organizer** (Xcode 11) **[NEW]**
- Accessed via Window > Organizer > Metrics tab; no SDK changes required
- Displays aggregated metrics from consented iOS 13+ device population

## Code Highlights

Using multiple XCTest metrics in a performance test:
```swift
func testApplyEffectPerformance() {
    measure(metrics: [XCTCPUMetric(), XCTMemoryMetric(), XCTStorageMetric()]) {
        // Apply image effect
        let result = applyEffect(to: sampleImage)
        XCTAssertNotNil(result)
    }
}
```

Adopting MetricKit in an app:
```swift
import MetricKit

class MyMetricSubscriber: NSObject, MXMetricManagerSubscriber {
    override init() {
        super.init()
        MXMetricManager.shared.add(self)
    }

    func didReceive(_ payloads: [MXMetricPayload]) {
        for payload in payloads {
            // Upload to your server or process on device
            if let data = payload.jsonRepresentation() {
                uploadToServer(data)
            }
        }
    }
}
```

Bookmarking a critical section with mxSignpost:
```swift
import MetricKit

let logHandle = MXMetricManager.makeLogHandle(category: "PhotoFeatures")

func savePhoto(_ photo: UIImage) {
    mxSignpost(.begin, log: logHandle, name: "SavePhoto")
    // ... save logic ...
    mxSignpost(.end, log: logHandle, name: "SavePhoto")
}
```

## Takeaways
- Add `XCTCPUMetric`, `XCTMemoryMetric`, and `XCTStorageMetric` to every existing `measure { }` test — it takes one line change and immediately catches multi-dimensional regressions in CI.
- Adopt MetricKit early (ideally before beta) so you have baseline field data to compare against once you ship; the 24-hour aggregation means you need days of usage before meaningful data accumulates.
- Use `mxSignpost` to instrument the 3–5 highest-impact features in your app; CPU-time data per region is far more actionable than app-level totals.
- On OLED devices, Dark Mode directly reduces the display battery subsystem — Average Pixel Luminance in MetricKit/Organizer will confirm the improvement after shipping Dark Mode support.

---
_Source: WWDC19 Session 417 page (abstract, chapter summaries, code samples, and resource links)._
