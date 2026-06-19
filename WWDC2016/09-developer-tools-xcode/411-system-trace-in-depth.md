# System Trace in Depth
**WWDC16 · Session 411** · [Watch](https://developer.apple.com/videos/play/wwdc2016/411/)

_Platforms:_ iOS 10, macOS Sierra 10.12, tvOS 10, watchOS 3

## Overview
This session from Instruments engineers Chad Woolf and Joe Grzywacz focuses on the System Trace Instruments template — the companion to Time Profiler for diagnosing second-order performance problems. While Time Profiler reveals where CPU cycles are spent when code is *running*, System Trace reveals why code cannot run: lock contention, thread preemption, misprioritized work, and virtual memory faults. The session uses a real app (Graphasaurus 2, an iOS graphing tool) to demonstrate three progressively uncovered performance problems encountered while tuning Instruments 8 itself.

A key Instruments 8 improvement covered here is the new **Points of Interest instrument** and `kdebug_signpost` API, which allows developers to annotate traces with high-level application events so that dense kernel-level trace data can be correlated with meaningful app activity (table view updates, downloads, frame rendering, etc.).

## Key Topics

### System Trace Overview
- System Trace puts the kernel into a special tracing mode that records all scheduling activity, system calls, and virtual memory operations.
- **Windowed Mode** (new default in Instruments 8): retains the last ~5 seconds of data, allowing developers to reproduce a problem and then stop the recording without losing the relevant window.
- The template works across all four Apple platforms (iOS, macOS, tvOS, watchOS).

### Points of Interest Instrument and kdebug Signposts
- New in Instruments 8: a blank-canvas instrument that plots developer-supplied events on the timeline.
- **Point events** ("lollipops"): `kdebug_signpost(code, arg1, arg2, arg3, arg4)` — marks an instant in time.
- **Region events** (intervals): `kdebug_signpost_start(code, arg1, ...)` + `kdebug_signpost_end(code, arg1, ...)` — marks a duration.
- `code` is an integer 0–16383 used to identify the signpost type.
- Pairing rules in Instruments configuration: by code only, code + first argument, or code + thread — allows parallel regions to be disambiguated.
- Regions can be colored using the fourth argument (`0`–`4` maps to five colors) by enabling "use last argument for color" in configuration.
- Old direct-system-call approach is deprecated; `kdebug_signpost` functions work from C, Objective-C, C++, and Swift.
- Custom templates can be saved (File > Save As Template) to preserve instrument configuration across sessions.

### Lock Contention (Case Study 1)
- Symptom: rendering frames took ~28 ms average; threads spending time in `blocked` and `runnable` states not visible in Time Profiler.
- Root cause: `NSAttributedString` creation acquires an internal lock; two dispatch worker threads creating attributed strings in parallel contended on the same lock.
- System Trace revealed: thread spending 109 µs to acquire a lock then 98 µs in `runnable` state waiting to get back on CPU — overhead nearly double the lock hold time.
- Fix: cache attributed strings ahead of time; look up in a dictionary lock-free. Result: threads run at 100% utilization, achieving near-linear multicore scaling.
- Thread State Summary view: shows total time spent per thread state (running, blocked, runnable, preempted, interrupted).

### Misprioritized Work (Case Study 2)
- Symptom after fix 1: still dropping frames; User Interactive Load Average shows orange bars (active user-interactive threads exceeding core count).
- Root cause: tooltip-generation queue created with `.userInteractive` QoS — same priority as render work — causing CPU contention between equally prioritized threads.
- System Trace + System Load instrument revealed 3–4 user-interactive threads competing on a 2-core device.
- Fix: change tooltip-generation queue to `.utility` QoS. Result: render threads get CPU priority, tooltip threads run in slack time. Frame time dropped from ~19 ms max to ~14.6 ms max; no dropped frames.

### Quality of Service Classes
- QoS is an attribute on blocks, queues, and threads expressing how urgently the work needs system resources.
- Classes constrain priority range *and* can throttle IO and CPU frequency.
- `userInteractive` → priority high 30s–40s; `utility` → priority ~4.
- Pick QoS that matches the actual urgency of the work; misuse causes CPU starvation of higher-priority work.

### Preemption and Interrupted State
- Involuntary preemption: a higher-priority thread needs the CPU; the OS removes your thread.
- Voluntary preemption: a spin lock calls `thread_switch` to yield its time quantum to the lock holder.
- Interrupted state: a hardware interrupt suspends the thread briefly (usually a few microseconds, rarely a problem).
- Thread Narrative view labels these states explicitly.

### Virtual Memory Faults
- Faults occur on first access to a page, not at allocation time.
- Resolved inline — the kernel handles the fault and resumes the thread transparently.
- Appear as blue capsules in Thread Strategy view; Thread Narrative includes backtrace of fault site.
- Virtual Memory instrument shows aggregate fault counts by type (zero fill, copy-on-write, etc.).
- Strategies: (a) budget enough slack that faults don't breach deadlines; (b) pre-touch pages on a background thread before the render thread needs them (only touch pages you will actually use).

## APIs & Frameworks

- **Instruments 8** [NEW] — redesigned System Trace template, Windowed Mode by default, Points of Interest instrument
- **Points of Interest instrument** [NEW in Instruments 8] — plots `kdebug_signpost` events on the timeline
- `kdebug_signpost(code: UInt32, arg1: UInt, arg2: UInt, arg3: UInt, arg4: UInt)` [NEW / replaces direct syscall] — point signpost
- `kdebug_signpost_start(code: UInt32, arg1: UInt, arg2: UInt, arg3: UInt, arg4: UInt)` [NEW] — begin region
- `kdebug_signpost_end(code: UInt32, arg1: UInt, arg2: UInt, arg3: UInt, arg4: UInt)` [NEW] — end region
- **System Load instrument** [NEW in Instruments 8] — User Interactive Load Average graph (10 ms buckets, orange when load > core count); shows active thread list at blue inspection head
- **Thread Strategy** view — per-thread scheduling timeline with state colors (running, blocked, runnable, preempted, interrupted)
- **Thread Narrative** view — chronological story of a thread's system calls and state transitions with backtraces
- **Thread Summary** view — aggregate time per thread state within a time filter selection
- **KDebug Interval Signpost by Code** table — aggregate statistics (count, min, mean, max, std dev) per signpost code
- **Time Filter** — `Control-click > Set Time Filter` on a detail row; grays out data outside selection in both detail and track views
- `DispatchQueue(label:qos:attributes:)` — used to set `DispatchQoS` on a queue; `.userInteractive`, `.utility`, `.background`, etc.
- `DispatchQoS` — `.userInteractive`, `.userInitiated`, `.default`, `.utility`, `.background`, `.unspecified`
- `CADisplayLink` — 60 Hz animation timer; used as signpost anchor in demo
- `ulock_wait` / `ulock_wake` — kernel primitives for `os_unfair_lock` and Swift concurrency primitives; visible in Thread Narrative
- `thread_switch` — voluntary yield system call visible in preemption narrative

## Code Highlights

Adding a point signpost (Swift):
```swift
// Mark a discrete event in the Points of Interest timeline
kdebug_signpost(5, 0, 0, 0, 0)  // code 5 = "Mouse Down" after configuration
```

Adding a region signpost with color coding:
```swift
// Color: 1 = green (success), 2 = red (error)
kdebug_signpost_start(2, taskPtr, 0, 0, 1)  // code 2, arg1 = task ID for pairing
// ... do work ...
kdebug_signpost_end(2, taskPtr, 0, 0, 2)    // 4th arg = color
```

Fixing QoS misprioritization:
```swift
// Before: competes with render work
let tooltipQueue = DispatchQueue(label: "tooltips", qos: .userInteractive, attributes: .concurrent)
// After: yields CPU to user-interactive work
let tooltipQueue = DispatchQueue(label: "tooltips", qos: .utility, attributes: .concurrent)
```

## Takeaways
- Time Profiler shows only what runs on the CPU; System Trace reveals the time *between* runs — lock contention, preemption, and VM faults can dwarf the time your code actually spends computing.
- Use `kdebug_signpost` to annotate traces with high-level app events; the Points of Interest instrument makes it practical to correlate dense kernel event streams with meaningful application phases like "render frame" or "download task."
- Misprioritized work (giving background tasks the same `.userInteractive` QoS as foreground rendering) wastes CPU time that belongs to your render loop; the System Load instrument's orange bars are a clear indicator.
- Virtual memory faults are resolved inline and can be pre-touched on background threads to avoid stutter; only pre-touch pages you will actually use.

---
_Source: WWDC16 Session 411 page (abstract, transcript, and resource links)._
