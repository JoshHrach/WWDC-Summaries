# Metal Game Performance Optimization
**WWDC18 · Session 612** · [Watch](https://developer.apple.com/videos/play/wwdc2018/612/)

_Platforms:_ iOS 12, macOS Mojave 10.14, tvOS 12

## Overview
This session tackles the most common performance issues found in top iOS Metal-based games, identified after Apple engineers profiled a large set of real games. The talk is organized around four root causes of frame-rate problems: inconsistent frame pacing (micro-stuttering), thread priority misconfiguration, thermal state mismanagement, and unnecessary GPU work.

A newly introduced Game Performance Template in Instruments 10 combines System Trace, Time Profiler, and Metal System Trace into a single pre-configured starting point for game profiling. A complementary new tool — the Dependency Viewer in the Metal Frame Debugger — provides a per-frame, per-pass breakdown of the GPU workload in a visual node-and-edge graph.

The session emphasizes that all the issues covered can be found in minutes using the available tools, and that profiling early and often is the single most important practice for shipping a high-performance game.

## Key Topics

### Frame Pacing (Micro-Stuttering)
- Micro-stuttering occurs when frame time exceeds the display refresh interval (e.g., 25 ms render time vs. 16.6 ms vsync)
- Presenting drawables "as soon as possible" without a minimum duration floor causes inconsistent frame presentation
- Identifying stutters via the Display track in Metal System Trace (long display intervals flagged with hints)
- Fix: use `MTLDrawable.present(afterMinimumDuration:)` or `present(at:)` to enforce a consistent frame floor

### Thread Priorities and Priority Decay
- iOS will decay the priority of busy threads over time to allow system work (e.g., App Store updates) to run
- Priority inversion: render thread depending on a lower-priority worker thread causes unintended stalls
- New Thread States view in Instruments shows preempted, blocked, and running states color-coded per thread
- Fix: configure the render pthread at fixed priority 45 and opt out of the scheduler's quality-of-service system

### Thermal State Management
- iOS devices manage thermals automatically; apps must respond gracefully to `NSProcessInfo.thermalState` changes
- Low Power Mode has similar effects and should be monitored via `NSProcessInfo.isLowPowerModeEnabled`
- Strategies for reducing thermal load: lower resolution of intermediate render targets, simplify shadow maps, remove post-processes, reduce target frame rate

### Unnecessary GPU Work — Dependency Viewer
- New Metal Frame Debugger feature: Dependency Viewer (node-edge graph of a single frame's render/blit/compute passes) **[NEW]**
- Nodes contain pass name, type icon (render/blit/compute), statistics, and thumbnails of written resources
- Edges show data dependencies between passes
- Helps identify non-obvious costs such as multiple cascading shadow map passes created by a single engine property toggle
- Labeling all passes with debug labels is critical for readability in every GPU tool

## APIs & Frameworks

**Metal (`Metal` framework)**
- `MTLDrawable.addPresentedHandler(_:)` — callback after drawable is displayed on screen
- `MTLDrawable.present(afterMinimumDuration:)` **[NEW]** — enforce minimum frame duration for consistent pacing
- `MTLDrawable.present(at:)` — present drawable at a specific host time
- `MTLCommandBuffer` GPU start/end time properties — measure actual GPU time consumed per command buffer

**Foundation / System**
- `NSProcessInfo.thermalState` — query current `ProcessInfo.ThermalState` (`.nominal`, `.fair`, `.serious`, `.critical`)
- `NSProcessInfoThermalStateDidChangeNotification` — observe thermal state changes
- `NSProcessInfo.isLowPowerModeEnabled` — query low power mode state
- `NSProcessInfoPowerStateDidChangeNotification` — observe power mode changes

**POSIX Threads**
- `pthread_attr_set_qos_class_np` — opt out of QoS-based priority decay
- `pthread_attr_setschedpolicy` / `pthread_attr_setschedparam` — set fixed thread priority (recommended: 45)

**Instruments 10**
- **Game Performance Template** **[NEW]** — pre-configured combination of System Trace, Time Profiler, and Metal System Trace
- **Thread States view** **[NEW]** — per-thread state visualization (running, preempted, blocked) with per-CPU-core priority color coding
- **Metal System Trace** — GPU timeline with vertex, fragment, and compute tracks plus Display track
- Windowed (Last N Seconds) recording mode — for profiling long game sessions

**Metal Frame Debugger (Xcode)**
- **Dependency Viewer / Dependency Graph** **[NEW]** — visual frame graph showing pass nodes and resource dependencies

## Code Highlights

Fixing frame pacing with a minimum duration:
```swift
// After encoding the final render pass:
let targetDuration: CFTimeInterval = 1.0 / 30.0  // targeting 30 fps
drawable.present(afterMinimumDuration: targetDuration)
commandBuffer.commit()
```

Configuring a high-priority render pthread (C):
```c
pthread_attr_t attr;
pthread_attr_init(&attr);
pthread_attr_set_qos_class_np(&attr, QOS_CLASS_USER_INTERACTIVE, 0);
// Fixed priority 45, opt out of QoS decay:
pthread_attr_setschedpolicy(&attr, SCHED_RR);
struct sched_param param = { .sched_priority = 45 };
pthread_attr_setschedparam(&attr, &param);
pthread_create(&renderThread, &attr, renderThreadFunc, NULL);
```

Responding to thermal state changes:
```swift
NotificationCenter.default.addObserver(forName: .NSProcessInfoThermalStateDidChange,
                                        object: nil, queue: .main) { _ in
    switch ProcessInfo.processInfo.thermalState {
    case .nominal:  enableFullQuality()
    case .fair:     reduceShadowResolution()
    case .serious:  disablePostProcessing()
    case .critical: targetThirtyFPS()
    @unknown default: break
    }
}
```

## Takeaways
- Micro-stuttering is invisible to fps counters but obvious to users; always set a minimum frame duration via `present(afterMinimumDuration:)`.
- The render thread must be configured at a fixed high priority (45) and opted out of QoS decay — this is a two-line code change with major impact.
- Thermal and power-state notifications are the correct mechanism for adapting GPU workload; never use `usleep` on the render thread.
- The new Dependency Viewer makes the true cost of "one engine checkbox" changes visible — use it before shipping any new rendering technique.

---
_Source: WWDC18 Session 612 page (abstract, chapter summaries, code samples, and resource links)._
