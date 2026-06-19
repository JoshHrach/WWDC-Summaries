# What's New in MetricKit
**WWDC20 · Session 10081** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10081/)

_Platforms:_ iOS 14, iPadOS 14

## Overview
MetricKit 2.0 arrived in iOS 14 with significant additions: three new performance metrics (CPU instructions, scroll hitches, and application exit reasons) and an entirely new diagnostics subsystem. The diagnostics interface delivers targeted, actionable data — including backtraces — for hangs, crashes, CPU exceptions, and disk write exceptions, collected passively from production and TestFlight users without requiring device access.

MetricKit fills a critical gap for developers who cannot attach a profiler to real users in the field. The daily payload model aggregates anonymized data across the day and delivers it once, protecting user privacy while providing enough signal to identify and triage regressions. The new 2.0 diagnostics mirror the existing metrics delivery model, making adoption straightforward.

## Key Topics

### New Metrics in MetricKit 2.0
Three new metrics expand the analytical surface:
- **CPU Instructions** — cumulative retired instructions; hardware- and frequency-independent absolute measure of app CPU workload. Added to `MXCPUMetric`.
- **Scroll Hitches** — ratio of hitch time to total scroll time in `UIScrollView`-based interfaces. A hitch is a rendered frame that does not appear on screen at its expected time.
- **Application Exit Reasons** — daily summary of foreground and background exit reasons and counts, helping diagnose excessive launch counts, background terminations, and crashes.

### MetricKit Diagnostics (New in 2.0)
A parallel daily payload (`MXDiagnosticPayload`) is now delivered alongside the existing `MXMetricPayload`. Subscribing requires implementing one additional delegate method. Four diagnostic types are provided:
- **Hang diagnostics** (`MXHangDiagnostic`) — duration of main-thread unresponsiveness + main-thread backtraces
- **CPU exception diagnostics** (`MXCPUExceptionDiagnostic`) — CPU time consumed, total sampled time, backtraces of high-CPU threads (equivalent to Xcode Organizer energy logs)
- **Disk write exception diagnostics** (`MXDiskWriteExceptionDiagnostic`) — total bytes written triggering the 1 GB/day threshold, backtraces of offending threads
- **Crash diagnostics** (`MXCrashDiagnostic`) — exception info, termination reason, virtual memory region info (bad access), full backtrace

### MXCallStackTree
A new shared data class used in all diagnostics. Contains unsymbolicated backtraces with binary UUID, offset, name, and frame address — everything needed to symbolicate offline with tools like `atos`. Designed for off-device processing and portable across performance tools.

### Delivery Model
Diagnostics are aggregated passively throughout the day and delivered in a single daily payload at the same time as the metric payload. The diagnostic payload maps one-to-one with the companion metric payload, enabling correlation between a regression signal (e.g., a hang spike in `MXAppRunTimeMetric`) and its associated diagnostic backtrace.

## APIs & Frameworks

### MetricKit Framework
- `MXMetricManager` — shared manager; add/remove subscribers
- `MXMetricManagerSubscriber` — protocol with `didReceive(_ payload: [MXMetricPayload])`
- `MXMetricPayload` — daily metric payload container

#### New Metrics
- `MXCPUMetric.cumulativeCPUInstructions` **[NEW]** — CPU instructions retired (hardware-independent)
- `MXAnimationMetric.scrollHitchTestDuration` / hitch ratio **[NEW]** — UIScrollView hitch rate
- `MXAppExitMetric` **[NEW]** — foreground and background exit reason counts
  - `foregroundExitData: MXForegroundExitData`
  - `backgroundExitData: MXBackgroundExitData`

#### New Diagnostics Classes
- `MXDiagnosticPayload` **[NEW]** — daily diagnostic payload container
- `MXDiagnostic` **[NEW]** — base class for all diagnostics; contains app build version metadata
- `MXCallStackTree` **[NEW]** — unsymbolicated backtrace data; JSON-serializable
- `MXHangDiagnostic: MXDiagnostic` **[NEW]** — hang duration + main-thread call stack
- `MXCPUExceptionDiagnostic: MXDiagnostic` **[NEW]** — CPU time, sampled time, thread call stacks
- `MXDiskWriteExceptionDiagnostic: MXDiagnostic` **[NEW]** — write bytes, thread call stacks
- `MXCrashDiagnostic: MXDiagnostic` **[NEW]** — exception type, termination reason, VM region, call stack

#### Updated Subscriber Protocol
- `MXMetricManagerSubscriber.didReceive(_ payload: [MXDiagnosticPayload])` **[NEW]** — opt in to diagnostics by implementing this method

#### Existing Metric Classes (referenced)
- `MXCPUMetric` — CPU time metrics
- `MXAppRunTimeMetric` — launch counts, resume counts, hang durations
- `MXMemoryMetric`, `MXDiskIOMetric`, `MXNetworkTransferMetric`, `MXDisplayMetric`

## Code Highlights

Setting up a MetricKit subscriber:
```swift
import MetricKit

class MySubscriber: NSObject, MXMetricManagerSubscriber {
    var metricManager: MXMetricManager?

    override init() {
        super.init()
        metricManager = MXMetricManager.shared
        metricManager?.add(self)
    }

    deinit {
        metricManager?.remove(self)
    }

    func didReceive(_ payload: [MXMetricPayload]) {
        for metricPayload in payload {
            // Process metrics
        }
    }
}
```

Adopting MetricKit Diagnostics (add alongside existing metric handler):
```swift
func didReceive(_ payload: [MXDiagnosticPayload]) {
    for diagnosticPayload in payload {
        // Consume diagnosticPayload
    }
}
```

## Takeaways

- MetricKit 2.0 adds CPU instructions (a hardware-independent CPU workload measure), scroll hitch rates, and application exit reasons to the existing metric set.
- The new diagnostics subsystem delivers hang, crash, CPU exception, and disk write exception backtraces passively from production users — no profiler attachment required.
- Diagnostics adoption requires only one new delegate method; the delivery model and JSON serialization are identical to the existing metrics pipeline.
- `MXCallStackTree` backtraces are unsymbolicated and must be post-processed offline with `atos` or equivalent tools using the binary UUID, offset, and frame address provided.

---
_Source: WWDC20 Session 10081 page (abstract, transcript, code samples, and resource links)._
