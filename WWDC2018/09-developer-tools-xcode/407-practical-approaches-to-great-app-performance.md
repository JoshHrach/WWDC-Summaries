# Practical Approaches to Great App Performance
**WWDC18 · Session 407** · [Watch](https://developer.apple.com/videos/play/wwdc2018/407/)

_Platforms:_ iOS 12, macOS Mojave 10.14

## Overview
This information-dense session shares practical, experience-driven strategies for identifying and resolving performance problems in real apps. The content is grounded in Apple's own work tuning first-party apps including Xcode and Photos for iOS, and covers both the methodological approach to performance work and specific tool-based techniques using Instruments and other diagnostics.

The session teaches a systematic, measurement-first approach to performance: identify what the user experiences, reproduce it, profile it with appropriate tools, fix one thing at a time, and verify. It emphasizes that performance work must happen continuously throughout development — not bolted on at the end — and provides concrete guidance on applying Instruments effectively.

## Key Topics

### Performance Methodology
- Measure before optimizing — never guess at the root cause
- Use Instruments' Time Profiler, System Trace, and os_signpost (Session 405) to understand what is happening
- Establish a reproducible scenario or test case for the performance problem
- Fix one issue at a time; re-measure after each change to verify improvement and avoid masking other problems
- Performance testing must cover a range of devices, including older and slower hardware

### Launch Time Optimization
- App launch is measured from when the user taps the icon to when the first frame is interactive
- Pre-main time: dylib loading, Objective-C class registration, `+load` methods; minimize work in `+load`
- Post-main time: `application(_:didFinishLaunchingWithOptions:)` and initial view controller setup
- Defer expensive initialization until it is first needed; use lazy properties
- Avoid synchronous disk I/O or network access on the main thread during launch

### Scrolling and Animation
- Scrolling must maintain 60 fps (16.6 ms per frame) — any work that exceeds this budget causes dropped frames
- Offload work from the main thread using `DispatchQueue` and `OperationQueue`; do not block the main thread
- Avoid expensive operations during cell dequeue: pre-compute layout metrics, use background image decoding
- Core Animation Instruments: identify offscreen rendering, blending, and rasterization costs
- Test scrolling performance with the Core Animation instrument in Instruments; look for frame rate drops

### Responsiveness and Hangs
- The main thread must remain unblocked to process user input and draw frames
- Synchronous operations that block the main thread cause hitches and apparent hangs
- Use Instruments' System Trace to identify what is blocking the main thread (I/O, locks, IPC)
- Move blocking work off the main thread; use async APIs

### Memory and Resource Management
- High memory pressure causes the system to terminate background apps and eventually the foreground app
- Use Instruments' Leaks, Allocations, and VM Tracker instruments
- Avoid retaining large objects longer than needed; cache strategically, not indefinitely
- Use `NSCache` instead of plain dictionaries for caches; it respects memory pressure automatically

### I/O and Disk Performance
- Synchronous disk reads on the main thread are a common cause of hangs
- Use background queues for file reads; use asynchronous APIs where available
- Avoid repeatedly reading the same file; cache results appropriately in memory
- Consider file format: binary plists are faster to parse than XML plists; prefer Codable with binary formats for large data sets

### Profiling with Instruments
- **Time Profiler**: identify which call stacks consume the most CPU time
- **System Trace**: understand thread state, system calls, IPC, and what is preempting your threads
- **Core Animation**: identify rendering bottlenecks (off-screen rendering, color blending, misaligned images)
- **Allocations**: track heap memory allocations and persistent object growth
- **os_signpost + Points of Interest**: mark and measure custom intervals of work (see Session 405)

## APIs & Frameworks

- **Instruments** — Time Profiler, System Trace, Core Animation, Allocations, Leaks, VM Tracker instruments
- **os_signpost** — signpost API for measuring custom intervals; integrates with Instruments (Session 405)
- **Grand Central Dispatch** (`libdispatch`) — `DispatchQueue`, `DispatchQueue.global()`, `DispatchQueue.main`
- **OperationQueue** — higher-level concurrency abstraction with dependency support
- **NSCache** — memory-pressure-aware cache replacing plain `Dictionary` for cached resources
- **`UITableView`** / **`UICollectionView`** — cell reuse patterns; `prefetchDataSource` for pre-loading
- **Core Animation** — layer rendering pipeline; `CALayer.shouldRasterize`, `CALayer.allowsGroupOpacity`
- **`+load` vs. `+initialize`** — prefer `+initialize` for Objective-C class initialization to reduce pre-main time
- **`@objc lazy var`** / **Swift `lazy` properties** — defer expensive initialization until first use

## Code Highlights

Lazy initialization to defer expensive setup:
```swift
class MyViewController: UIViewController {
    private lazy var expensiveResource: DataModel = {
        return DataModel(loadingFrom: Bundle.main)
    }()
}
```

Moving disk I/O off the main thread:
```swift
DispatchQueue.global(qos: .userInitiated).async {
    let data = try? Data(contentsOf: fileURL)
    DispatchQueue.main.async {
        self.processData(data)
    }
}
```

Using NSCache for memory-pressure-aware caching:
```swift
let imageCache = NSCache<NSString, UIImage>()
imageCache.countLimit = 100
imageCache.totalCostLimit = 50 * 1024 * 1024  // 50 MB
```

## Takeaways
- Performance is a feature that must be built in from the start; use Instruments during every development phase, not just at ship time.
- The majority of app performance problems fall into three categories: main-thread blockage (I/O, locks), excessive memory use, and unnecessary work during scrolling — all identifiable with the standard Instruments suite.
- Deferring expensive initialization with lazy properties and offloading I/O to background queues are the two highest-impact techniques for improving launch time and scrolling responsiveness.
- Test on the oldest device in your deployment target — performance problems invisible on a new device are glaring on older hardware.

---
_Source: WWDC18 Session 407 page (abstract, related session links, and resource links)._
