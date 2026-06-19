# Profile, Fix, and Verify: Improve App Responsiveness with Instruments

**WWDC26 · Session 268** · [Watch](https://developer.apple.com/videos/play/wwdc2026/268/)

_Platforms:_ Xcode 27 · iOS · macOS · iPadOS · tvOS · watchOS · visionOS

## Overview

This session presents a systematic, four-step framework for diagnosing and resolving app responsiveness problems using Instruments 27. Rather than a general Instruments overview, the session focuses on a specific workflow — profile, fix, verify — and walks through three real hang scenarios from a note-taking app: a CPU-saturated main thread, execution contention on the Main Actor from concurrent tasks, and system-level blocking from synchronous file I/O.

The diagnostic flow follows four ordered layers: CPU saturation (is the CPU too busy?), sampling data visualization (what is it doing?), execution contention (are tasks competing for an actor?), and system blocking (is the thread blocked by the OS?). Each layer maps to a specific Instruments template: Time Profiler or the new Top Functions view for sampling, Swift Concurrency instrument for actor contention, and System Trace for system-level blocking.

Two new Instruments 27 capabilities receive detailed coverage: the Top Functions view (which instantly surfaces the most expensive functions from a CPU profile without requiring manual call-tree navigation) and the Inspector panel in System Trace (which explains why a thread is blocked with human-readable context). The session also demonstrates the `@concurrent` task attribute for moving work off the Main Actor and signpost intervals for narrowing the region of interest before profiling.

## Key Topics

### Diagnostic Flow (Four Layers)
1. **CPU saturation** — is the CPU actually busy? Use Activity Monitor / Instruments timeline.
2. **Sampling data visualization** — what code is consuming CPU? Use Time Profiler, Flame Graph, or Top Functions.
3. **Execution contention** — are Swift tasks competing for the Main Actor? Use Swift Concurrency instrument.
4. **System blocking** — is the thread blocked waiting for I/O or a lock? Use System Trace with the new Inspector panel.

### Sampling Data Visualization
- **Call Tree view** — traditional bottom-up weighted call tree.
- **Flame Graph** — proportional width visualization of the call stack.
- **[NEW] Top Functions view** — ranked list of the most CPU-expensive functions; eliminates manual call-tree diving for common regressions.
- **[NEW] Comparison mode** — compare two trace recordings side-by-side to verify a fix reduced cost.

### Execution Contention
- **Swift Concurrency instrument** — shows tasks and which actor/executor they run on.
- Thumbnail rendering tasks running on the Main Actor block UI updates.
- Fix: add `@concurrent` to the `Task` closure to move work off the main thread.

### System Blocking
- **System Trace** — kernel-level thread state visualization.
- **[NEW] Inspector panel** — human-readable explanation of why a thread is in the blocked state.
- Synchronous file I/O (`Data.write(to:options:)`) on the main thread causes the OS to block it.
- Fix: wrap file I/O in a `Task { @concurrent in ... }` to move it off the main thread.

### Signposts
- Use `OSSignposter` to annotate the region of interest before profiling, so Instruments can focus on the relevant time window.

## APIs & Frameworks

**Instruments (Xcode 27)**
- **[NEW]** Top Functions view — ranked CPU cost list from sampling data
- **[NEW]** Comparison mode — side-by-side trace comparison to verify fixes
- **[NEW]** Inspector panel in System Trace — human-readable thread-blocked explanation
- Time Profiler instrument — CPU sampling
- Flame Graph view — call stack proportional visualization
- Call Tree view — weighted bottom-up call tree
- Swift Concurrency instrument — actor/executor contention visualization
- System Trace instrument — kernel-level thread state (running, blocked, preempted)

**Swift Concurrency**
- `Task(name:priority:operation:)` — named task for Instruments identification
- **[NEW]** `@concurrent` attribute on `Task` closure — explicit off-main-actor execution
- `@MainActor` — annotation that binds a type or function to the main thread
- `await` — suspension point; shows as yield in Instruments

**os.signpost (Performance Instrumentation)**
- `OSSignposter(subsystem:category:)` — signpost logger instance
- `OSSignposter.beginInterval(_:)` → `OSSignpostIntervalState`
- `OSSignposter.endInterval(_:_:)` — ends a named interval
- `OSSignpostIntervalState` — opaque state handle for an active interval
- `.pointsOfInterest` category — surfaces intervals in Instruments timeline

**Foundation**
- `PropertyListEncoder` — binary plist serialization
- `Data.write(to:options:)` — synchronous file write (move off main thread)
- `.atomic` write option

**Swift Type System (Performance Patterns)**
- Existentials (`any Protocol`) vs. concrete types vs. generics (`some Protocol`) — existentials add dynamic dispatch overhead visible in Top Functions
- Prefer generics or concrete types over `any` for hot paths

**Related Documentation**
- [Analyzing CPU profiles with call tree views](https://developer.apple.com/documentation/Xcode/analyzing-cpu-profiles-with-call-tree-views)

## Code Highlights

Adding signpost intervals to narrow the profiling region:

```swift
import os.signpost

let signposter = OSSignposter(subsystem: "Demo App", category: .pointsOfInterest)
var lassoIntervalState: OSSignpostIntervalState? = nil

func lassoSelectionUpdated() {
    lassoIntervalState = signposter.beginInterval("Lasso Selection")
    // Update selection in canvas…
}

func lassoSelectionEnded() {
    // Finalize lasso selection...
    signposter.endInterval("Lasso Selection", lassoIntervalState!)
}
```

Moving thumbnail rendering off the Main Actor with `@concurrent`:

```swift
// Before: runs on Main Actor, blocks UI
thumbnail = await Task(name: "Render Thumbnail") {
    await renderThumbnail(drawingData: drawingData, canvasImages: canvasImages,
                          size: CGSize(width: 300, height: 240))
}.value

// After: @concurrent moves work off Main Actor
thumbnail = await Task(name: "Render Thumbnail") { @concurrent in
    await renderThumbnail(drawingData: drawingData, canvasImages: canvasImages,
                          size: CGSize(width: 300, height: 240))
}.value
```

Moving synchronous file I/O off the main thread:

```swift
// After: file write moved to background task
Task { @concurrent in
    let encoder = PropertyListEncoder()
    encoder.outputFormat = .binary
    guard let data = try? encoder.encode(snapshots) else { return }
    let id = signposter.beginInterval("Writing To File")
    try? data.write(to: fileURL, options: .atomic)
    signposter.endInterval("Writing To File", id)
}
```

## Takeaways

- The four-layer diagnostic framework (CPU saturation → sampling → contention → system blocking) prevents wasted time applying the wrong fix to the wrong problem class.
- Top Functions view eliminates the most tedious part of Instruments — manually collapsing call trees — for the most common regression scenario.
- `@concurrent` on `Task` closures is the simplest fix for Main Actor contention; use the Swift Concurrency instrument first to confirm contention before adding it.
- Synchronous I/O on the main thread is the leading cause of low-CPU hangs; System Trace + the new Inspector panel is the only reliable way to diagnose it without guessing.

---
_Source: WWDC26 Session 268 page (abstract, chapter summaries, code samples, and resource links)._
