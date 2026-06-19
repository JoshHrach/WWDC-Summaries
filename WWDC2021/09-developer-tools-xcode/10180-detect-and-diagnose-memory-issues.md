# Detect and diagnose memory issues
**WWDC21 · Session 10180** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10180/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, watchOS 8, tvOS 15

## Overview
This session provides a systematic workflow for finding, diagnosing, and fixing memory problems in Apple platform apps, anchored around Xcode 13's new ability to automatically collect memgraph diagnostics from failing performance XCTests. The speakers cover memory footprint composition (dirty, compressed, clean), three major memory issue types (leaks, heap allocation regressions, fragmentation), and the command-line tools used to triage each.

The workflow is: write a performance `XCTMemoryMetric` test, set a baseline, let the test catch regressions, and then use automatically collected memgraph files with command-line tools to pinpoint the root cause. Xcode 13 now enables malloc stack logging (MSL) automatically in XCTest, providing allocation backtraces for every object without any manual setup.

## Key Topics

### Memory Footprint Composition
- **Dirty memory**: written by the app — heap allocations, decoded image buffers, framework data.
- **Compressed memory**: dirty pages the memory compressor has compressed (decompressed on access); iOS has no swap.
- **Clean memory**: unwritten pages or memory-mappable data (on-disk files, framework text); does not count against footprint.
- Footprint = dirty + compressed.

### Why Footprint Matters
- Low footprint allows apps to stay resident in background and activate faster.
- Reduces peak memory pressure, preventing OS termination.
- Enables more features (video, animations) within the memory budget.
- Improves experience on older/lower-memory devices.

### XCTest Performance Workflow
- Use `measure(metrics:options:block:)` with `XCTMemoryMetric(application:)` to measure per-test memory.
- Set a baseline after the first run; subsequent runs fail if average exceeds baseline (regression).
- Xcode 13: enable `enablePerformanceTestsDiagnostics` flag with `xcodebuild` to auto-collect ktrace and memgraph files on test failure.
- ktrace files → opened in Instruments for general system investigations.
- Memgraphs → used with Xcode's visual memory debugger and command-line tools.
- Two memgraphs per failing test: `pre_` (start of iteration) and `post_` (end) to measure growth over one iteration.
- MSL is automatically enabled in XCTest memgraphs, providing full allocation backtraces.

### Memory Leaks
- A leak occurs when an object is allocated, all references are lost, but the object is never freed.
- Common cause in Swift: retain cycles (strong circular references between two objects).
- Fix: break cycles with `weak` references; avoid strong circular references.
- If working with `UnsafePointer` / raw allocations (not ARC), manually `deallocate()` before losing the reference.
- Diagnosis: `leaks <memgraph>` — reports leaked objects, ROOT CYCLE indicators, and MSL allocation backtraces.

### Heap Allocation Regressions
- An increase in live heap objects causing footprint growth.
- Strategies: remove unused allocations, shrink oversized buffers, set objects to `nil` when done.
- Diagnosis workflow:
  1. `vmmap -summary <post.memgraph>` — see footprint breakdown by region; look at `MALLOC_*` regions.
  2. `heap -diffFrom <pre.memgraph> <post.memgraph>` — shows objects present in post but not pre, grouped by class.
  3. `heap -addresses <post.memgraph> -type non-object -minimumSize 500000` — get addresses of large raw allocations.
  4. `leaks --traceTree <address> <memgraph>` — reference tree of objects pointing to a specific address (useful without MSL).
  5. `leaks --referenceTree <memgraph>` (with `--groupByType`) — top-down reference tree to find aggregated memory.
  6. `malloc_history -fullStacks <memgraph> <address>` — full allocation backtrace for a specific address (requires MSL).

### Heap Fragmentation
- Occurs when pages are partially used: deallocated objects leave free slots too small or non-contiguous for future large allocations.
- A fragmentation multiplier: 50% fragmentation doubles dirty page count.
- Target: less than 25% fragmentation.
- Reduce by allocating objects with similar lifetimes close together in memory; use autorelease pools to ensure co-located objects are released together.
- Long-running processes (extensions) are especially prone.
- Diagnosis: `vmmap -summary` → bottom section → `% FRAG` and `dirty+swap frag size` columns per malloc zone.
- With MSL enabled, heap objects land in `MallocStackLoggingLiteZone` — focus on that zone's fragmentation %.
- Use the Allocations track in Instruments to see persisted vs. destroyed objects in a time range.

## APIs & Frameworks

**XCTest**
- `XCTMemoryMetric(application:)` — measures process memory footprint during a test **[existing]**
- `measure(metrics:options:block:)` — performance measurement harness with per-metric baselines **[existing]**
- `XCTMeasureOptions` with `.manuallyStart` invocation option — defers measurement start until `startMeasuring()` is called **[existing]**
- `XCTAssertTrue(_:)`, `waitForExistence(timeout:)` — UI test assertions **[existing]**
- Automatic malloc stack logging (MSL) in XCTest memgraphs **[NEW in Xcode 13]**
- Automatic ktrace and memgraph collection on performance test failure via `enablePerformanceTestsDiagnostics` **[NEW in Xcode 13]**

**MetricKit**
- `MXMemoryMetric` — production memory metrics from user devices **[existing]**

**Xcode Organizer**
- Memory metrics from shipping apps **[existing]**

**Command-Line Tools**
- `leaks <memgraph>` — detects leaked objects and retain cycles, reports MSL backtraces **[existing]**
- `leaks --traceTree <address> <memgraph>` — reference tree for a specific object **[existing]**
- `leaks --referenceTree [--groupByType] <memgraph>` — top-down memory reference tree **[existing]**
- `vmmap -summary <memgraph>` — memory region breakdown with dirty/compressed sizes and fragmentation **[existing]**
- `heap -diffFrom <pre.memgraph> <post.memgraph>` — heap object diff between two snapshots **[existing]**
- `heap -addresses <memgraph> -type <type> -minimumSize <bytes>` — addresses of objects of a given type/size **[existing]**
- `malloc_history -fullStacks <memgraph> <address>` — full allocation backtrace for an address **[existing]**

**Swift / ARC**
- `weak` reference modifier — breaks retain cycles without preventing deallocation **[existing]**
- Setting instance to `nil` — triggers ARC deallocation, releasing associated buffers **[existing]**
- Autorelease pools (`autoreleasepool {}`) — batch-release of objects to reduce fragmentation **[existing]**

## Code Highlights

Performance XCTest measuring memory:
```swift
func testSaveMeal() {
    let app = XCUIApplication()
    let options = XCTMeasureOptions()
    options.invocationOptions = [.manuallyStart]

    measure(metrics: [XCTMemoryMetric(application: app)],
            options: options) {
        app.launch()
        startMeasuring()
        app.cells.firstMatch.buttons["Save meal"].firstMatch.tap()
        let savedButton = app.cells.firstMatch.buttons["Saved"].firstMatch
        XCTAssertTrue(savedButton.waitForExistence(timeout: 30))
    }
}
```

Breaking a retain cycle with a weak reference:
```swift
class MenuItem {
    weak var mealPlan: MealPlan?  // was 'var mealPlan: MealPlan?' causing cycle
}
```

Releasing a large buffer when no longer needed:
```swift
func saveMeal() {
    // ... write mealData to disk ...
    mealData = nil  // releases backing buffer immediately
}
```

## Takeaways
- Use `XCTMemoryMetric` performance tests with baselines to automatically catch memory regressions before shipping; Xcode 13 collects memgraph diagnostics on failure automatically.
- The diagnostic triage order is: check for leaks first → then heap regressions (vmmap → heap diffFrom → malloc_history) → then fragmentation.
- Retain cycles are the most common Swift leak cause; use `weak` references to break them.
- Reduce heap regressions by setting references to `nil` when done and avoiding holding large buffers longer than needed; target fragmentation below 25%.

---
_Source: WWDC21 Session 10180 page (abstract, full transcript, code sample, and resource links)._
