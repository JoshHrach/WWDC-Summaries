# Understand and Eliminate Hangs from Your App
**WWDC21 · Session 10258** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10258/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session provides a deep-dive into app hangs — what they are, their common causes, the tools for diagnosing them, and concrete strategies for elimination. A hang is any period during which the main thread is unresponsive to user input, typically perceptible at delays over 250ms and always apparent at over one second. During a hang, input events are buffered rather than processed, causing them to compound.

The session categorizes hang causes into two groups: the main thread being too busy (proactive over-computation, irrelevant work dispatched onto it, suboptimal API) and the main thread being blocked (synchronous API, I/O, data store contention, synchronization primitives, expensive queries). Solutions include caching with `NSCache`, notification observation via `NSNotificationCenter`, asynchronous API alternatives, and GCD `dispatch_async` patterns.

Tools covered: Time Profiler and System Trace instruments for development, MetricKit for field hang diagnostics, and Xcode Organizer Hang Rate chart for version-over-version tracking.

## Key Topics

### What is a Hang?
- A hang occurs when the main runloop is blocked from processing events
- The main thread processes one event per runloop turn: receive event → process → update UI
- Buffered events compound: touches during a hang are queued, then all processed sequentially after the hang
- Threshold: delays >1 second always appear as hangs; shorter delays are context-dependent (0.5s during scroll is jarring; same delay entering a view is less noticeable)

### Causes: Main Thread Too Busy
1. **Proactive over-computation**: doing more than needed (e.g., loading all images when only 4 are visible)
2. **Irrelevant work via dispatch_sync**: main thread `dispatch_sync`-ing onto a low-priority queue forces it to wait for all pending blocks on that queue
3. **Work dispatched onto main queue**: blocks `dispatch_async`-ed from background queues still run on main thread
4. **Suboptimal API**: CPU-intensive operations (e.g., bitmap context for rounded corners) when GPU-based Core Animation alternatives are faster

### Causes: Main Thread Blocked
1. **Synchronous APIs**: network requests, file I/O that may take unbounded time
2. **File I/O**: latencies depend on hardware and concurrent access; avoid on main thread
3. **Non-concurrent data stores**: reads blocked by concurrent writes (e.g., SQLite single-writer)
4. **Synchronization primitives**: `@synchronized`, `dispatch_sync`, `os_unfair_lock`, POSIX locks — especially semaphores (no priority propagation)
5. **Anti-pattern**: semaphore used to make async API synchronous (`semaphore.wait()` on main thread)
6. **Expensive queries**: fetching slowly-changing values on every user interaction (e.g., querying all contacts on each tap)

### Hardware and System State
- Device condition (CPU load, storage speed, memory pressure) significantly affects hang frequency in the field
- Test on the oldest supported hardware as a performance benchmark

### Tools for Diagnosing Hangs

**Instruments — Time Profiler**
- Shows app callstacks aggregated over time during the hang
- Identifies exact code paths responsible for blocking

**Instruments — System Trace**
- Adds system call data, VM faults, I/O, inter-process communication context
- Red line: system calls; purple: VM faults; horizontal blue bar: main thread busy

**MetricKit — Hang Diagnostics**
- `MXHangDiagnostic` delivered in `MXDiagnosticPayload` via `MXMetricManagerSubscriber.didReceive(_:)`
- Provides aggregated call trees from hangs occurring in the field
- Enables prioritization by which hangs affect most customers

**Xcode Organizer — Hang Rate Chart**
- Shows hang rate per app version for trend analysis and regression identification

### Strategies for Eliminating Hangs

**Caching**
- `NSCache`: in-memory store for expensive-to-generate assets (e.g., formatted image tiles)
- Cache invalidation must happen asynchronously on a secondary queue
- Avoids repeated expensive computation; replaces heavy work with O(1) memory read

**Notification Observers**
- `NotificationCenter.addObserver(forName:object:queue:using:)`: react to state changes without polling
- `NSNotification.Name.ABDatabaseChangedExternally` example: update contacts cache when database changes
- Dispatch handler `async` to secondary queue to keep main thread free
- Caution: coalesce or filter high-frequency notifications to avoid CPU churn

**Asynchronous API Alternatives**
- Use `URLSession.dataTask` instead of synchronous networking
- Use asynchronous file/data reading APIs instead of synchronous counterparts
- Async APIs indicated by "asynchronously" in name or presence of completion handler

**Grand Central Dispatch**
- `DispatchQueue.global().async { ... }`: move work to background thread
- Dispatch back to main: `DispatchQueue.main.async { ... }` inside the async block
- Pre-warming: `dispatch_async` onto a prefetch queue while main thread does other work; `dispatch_sync` onto it later when results are needed
- `DispatchQueue.init(label:)` with serial queue + `sync` for ordered synchronization when needed

## APIs & Frameworks

- `Dispatch` framework
- `DispatchQueue`
- `DispatchQueue.global(qos:)`
- `DispatchQueue.main`
- `DispatchQueue.async(execute:)`
- `DispatchQueue.sync(execute:)`
- `Foundation` framework
- `NSCache`
- `NSCache.setObject(_:forKey:)`
- `NotificationCenter`
- `NotificationCenter.addObserver(forName:object:queue:using:)`
- `NSNotification.Name`
- `URLSession` (async methods)
- `os_unfair_lock`
- `MetricKit` framework
- `MXMetricManagerSubscriber`
- `MXDiagnosticPayload`
- `MXHangDiagnostic`
- Instruments (Time Profiler, System Trace — tooling, no public API)
- Xcode Organizer Hang Rate (tooling, no public API)
- Core Animation (CALayer for GPU-based rounded corners — preferred over CPU bitmap context)
- `CALayer.cornerRadius`
- `CALayer.masksToBounds`

## Code Highlights

Pattern: async background work with main thread completion:
```swift
DispatchQueue.global(qos: .userInitiated).async {
    let result = expensiveComputation()
    DispatchQueue.main.async {
        self.updateUI(with: result)
    }
}
```

Notification observer for rarely-changing state (contacts example):
```swift
NotificationCenter.default.addObserver(
    forName: NSNotification.Name.ABDatabaseChangedExternally,
    object: nil,
    queue: nil
) { [weak self] _ in
    DispatchQueue.global(qos: .background).async {
        self?.refreshContactsCache()
    }
}
```

GPU-based rounded corners instead of bitmap context:
```swift
imageView.layer.cornerRadius = 8
imageView.layer.masksToBounds = true
// No UIBezierPath/bitmap context needed
```

## Takeaways

- Any synchronous operation on the main thread that may take unbounded time (networking, file I/O, lock waiting, expensive queries) is a hang waiting to happen; always use async alternatives.
- `NSCache` is the simplest fix for hangs caused by repeatedly computing the same expensive asset; cache invalidation must be async.
- The `dispatch_async` + completion-handler-back-to-main pattern is the fundamental GCD fix for most hang causes; use it for any work that doesn't need to immediately block UI.
- Use Time Profiler and MetricKit together: Time Profiler for development-time hangs, MetricKit `MXHangDiagnostic` for field prioritization of issues customers actually hit.

---
_Source: WWDC21 Session 10258 page (abstract, chapter summaries, code samples, and resource links)._
