# Optimizing App Launch
**WWDC19 · Session 423** · [Watch](https://developer.apple.com/videos/play/wwdc2019/423/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
This session provides a comprehensive guide to understanding, measuring, and improving app launch performance. It covers what launch actually is (six distinct phases from dynamic linker through first-frame render), how to set up a clean and reproducible measurement environment, how to use the new App Launch Instrument template in Xcode 11/Instruments, and how to monitor launch continuously with XCTest performance tests and MetricKit.

A live Instruments demo traces a sample app ("Star Searcher") from a 2.5-second cold launch to a 300ms warm launch by diagnosing three specific problems: a third-party framework doing 375ms of work in a `+load` static initializer, a priority inversion caused by using a `DispatchSemaphore` to block the main thread while a background-QoS queue fetched data, and speculative pre-warming of view controllers in `cellForRowAt` that was adding 880ms to the first-frame render phase. The session also surfaces major iOS 13 system-level improvements — dyld3 now caches third-party app dependencies, the scheduler is priority-aware, and Auto Layout and Objective-C runtime are faster.

## Key Topics
- **Launch types** — Cold (app not in memory; most variable), Warm (app partially in memory; more consistent — use for measurements), Resume (app already running; not a launch)
- **Goal** — render first frame within 400ms so pixels appear during the launch animation; app must be interactive by animation end
- **Six launch phases**:
  1. `dyld` — loads shared libraries/frameworks; dyld3 now caches third-party app dependencies **[NEW iOS 13]**
  2. `libSystemInit` — fixed-cost system component initialization
  3. Static runtime init — `+load` methods and C++ static constructors from app code and linked frameworks
  4. UIKit init — `UIApplication` + `UIApplicationDelegate` instantiation
  5. App init — `application(_:didFinishLaunchingWithOptions:)` / `UISceneDelegate.scene(_:willConnectTo:options:)`
  6. First frame render — view creation, Auto Layout, draw, commit
- **dyld3 for third-party apps** — dependency resolution cached across launches; avoid linking unused frameworks (hidden cost), avoid `dlopen`/`NSBundle.load` (forfeits cache benefit), hard-link all dependencies **[NEW iOS 13]**
- **Static initializers** — identify via Instruments; move work from `+load` (runs every launch) to `+initialize` (lazy, first method call); audit third-party framework costs
- **Priority inversion pattern** — using `DispatchSemaphore.wait()` on main thread (User Interactive) to block on a background-QoS queue; fix: use `queue.sync { }` so GCD propagates main-thread priority to the worker
- **Data loading discipline** — load only the rows needed for the first frame synchronously; load the rest asynchronously and update UI after launch
- **Lazy view controller creation** — never pre-warm view controllers for future screens in `cellForRowAt`; defer to `didSelectRowAt`
- **Measurement environment** — reboot + settle, airplane mode or mock network, static iCloud account or log out, release build, warm launches only, fixed consistent test data set, same device set across sessions
- **XCTest app launch measurement** — `measure(metrics:block:)` with `XCTApplicationLaunchMetric`; 1 throwaway + 5 measured launches → statistical result **[NEW Xcode 11]**
- **MetricKit** — on-device performance/battery metric collection with 24-hour aggregation, delivered via `MXMetricManagerSubscriber.didReceive(_:)` **[NEW iOS 13]**
- **Xcode Organizer** — aggregated launch/battery histograms from opted-in users, by software version and device **[NEW Xcode 11]**

## APIs & Frameworks

### Instruments (NEW)
- **App Launch template** **[NEW Xcode 11]** — purpose-built template for launch profiling; shows all six launch phases color-coded; displays per-thread state (running/blocked/preempted/runnable) with CPU sample stack traces
- Thread state colors: blue = running, gray = blocked, red = runnable (no CPU), orange = preempted
- Wall clock vs. CPU clock — wall clock includes profiling overhead; CPU clock reflects actual app CPU cost; distinguish when comparing phases
- Aggregated call tree — symbols ordered by CPU sample count; highlights app-owned symbols in blue

### XCTest (NEW)
- `XCTApplicationLaunchMetric` **[NEW Xcode 11]** — measures time from process creation to first frame; use with `measure(metrics:block:)`
- `measure(metrics: [XCTMetric], options: XCTMeasureOptions, block: () -> Void)` — runs 1 warm-up + `iterationCount` (default 5) measured launches
- `XCTMeasureOptions` — `iterationCount`, `invocationOptions`
- `startMeasuring()` / `stopMeasuring()` — for precise measurement windows within a test

### MetricKit (NEW iOS 13)
- `MXMetricManager` — `add(_:)` subscriber, `makeLogHandle(category:)` for custom signposts
- `MXMetricManagerSubscriber` — `didReceive(_ payloads: [MXMetricPayload])`
- `MXMetricPayload` — `applicationLaunchMetrics: MXAppLaunchMetric?`, `applicationResponsivenessMetrics`, `cpuMetrics`, `memoryMetrics`, `displayMetrics`, battery metrics; 24-hour aggregated
- `MXAppLaunchMetric` — `histogrammedTimeToFirstDraw: MXHistogram<UnitDuration>`

### Performance-Relevant Runtime APIs
- `os_signpost(.begin, log:, name:)` / `os_signpost(.end, ...)` — bracket app-defined phases within the extended launch phase; appears in Instruments timeline
- `DispatchQueue.sync { }` — correct primitive for priority-preserving synchronous work across queues (vs. semaphore)
- `class func initialize()` — Objective-C class lazy init; runs only on first method call (prefer over `+load`)
- dyld3 cache — automatic for system frameworks; extended to third-party apps in iOS 13; requires hard-linked (not dynamically loaded) dependencies

## Code Highlights

```swift
// XCTest app launch performance test (Xcode 11)
import XCTest

class LaunchTests: XCTestCase {
    func testLaunchPerformance() throws {
        measure(metrics: [XCTApplicationLaunchMetric()]) {
            XCUIApplication().launch()
        }
        // Runs 1 throwaway + 5 measured launches; reports mean, std dev, etc.
    }
}
```

```swift
// Fix priority inversion: replace DispatchSemaphore with queue.sync
// WRONG — blocks main thread; worker stays at background QoS
let sema = DispatchSemaphore(value: 0)
dataQueue.async {
    loadData()
    sema.signal()
}
sema.wait()  // main thread (User Interactive) blocked by background worker

// CORRECT — GCD propagates main thread priority to worker during sync
dataQueue.sync {
    loadData()
}
```

```swift
// Load only first-frame data synchronously; rest asynchronously
override func application(_ app: UIApplication,
                          didFinishLaunchingWithOptions opts: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    dataProvider.loadStarDataSync(rows: 0..<20)  // enough for first screen
    DispatchQueue.global(qos: .userInitiated).async {
        self.dataProvider.loadAllStarData()
        DispatchQueue.main.async { self.tableView?.reloadData() }
    }
    return true
}
```

```swift
// MetricKit subscription for field launch metrics
import MetricKit

class AppDelegate: UIResponder, UIApplicationDelegate, MXMetricManagerSubscriber {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions _: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        MXMetricManager.shared.add(self)
        return true
    }

    func didReceive(_ payloads: [MXMetricPayload]) {
        for payload in payloads {
            if let launchMetric = payload.applicationLaunchMetrics {
                // launchMetric.histogrammedTimeToFirstDraw — distribution of launch times
                print(launchMetric.histogrammedTimeToFirstDraw)
            }
        }
    }
}
```

## Takeaways
- dyld3 now caches runtime dependencies for third-party apps in iOS 13 — do not call `dlopen`/`NSBundle.load` at launch and do not link unused frameworks, or you forfeit the cache benefit.
- `+load` in third-party frameworks is the single most common hidden launch cost: use Instruments' Static Runtime Init phase to identify which frameworks call `+load` and quantify the cost before deciding whether the framework is worth it.
- Use `queue.sync { }` instead of a `DispatchSemaphore` when the main thread must wait for background work — GCD will temporarily boost the worker's QoS to match, eliminating priority inversion.
- Commit only the data and views needed for the first visible frame; defer everything else — pre-warming detail view controllers in `cellForRowAt` is a well-intentioned anti-pattern that consistently adds hundreds of milliseconds.
- Integrate `XCTApplicationLaunchMetric` into CI immediately and chart results over every build; the only reliable way to prevent launch regressions is automated measurement, not code review.

---
_Source: WWDC19 Session 423 page (full transcript, abstract, and resource links)._
