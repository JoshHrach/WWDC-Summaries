# Optimize App Power and Performance for Spatial Computing
**WWDC23 · Session 10100** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10100/)

_Platforms:_ visionOS 1

## Overview
This session establishes the performance model for visionOS and provides actionable optimization guidance across every major subsystem: rendering (SwiftUI/UIKit, RealityKit, Metal), user input, ARKit, audio/video playback, SharePlay, and memory/thermal pressure. Because the visionOS compositor renders continuously at 90+ fps even when apps produce no updates, performance and power efficiency are more critical on this platform than on any prior Apple platform.

The session outlines a three-phase workflow: profile on device using Instruments (especially RealityKit Trace) and Xcode Gauges during development; adapt to field data via MetricKit and Xcode Organizer after release; and build a proactive performance plan around visionOS-specific metrics before shipping.

## Key Topics

### Unique visionOS Performance Characteristics
- The compositor renders at 90+ Hz continuously — every body, hand, and eye movement triggers a display update regardless of app activity
- The system runs spatial algorithms (eye tracking, hand tracking, scene understanding) for all apps simultaneously
- Multiple apps run concurrently in Shared Space; your app competes for CPU, GPU, and memory with other foreground apps
- Thermal pressure replaces battery life as the primary power concern (device is tethered during use; heat affects comfort and sustainable performance)
- Even momentary main-thread hangs are perceptible and disruptive to the sense of immersion

### Rendering Pipeline on visionOS
1. App updates content on the main thread
2. Updates go to the **render server** (system process handling all apps and user input compositing)
3. Render server sends a completed frame to the **compositor**
4. Compositor continuously supplies frames to the display at the refresh rate

If the app's render work causes the render server to miss its deadline, the visual update is delayed by one full frame — making the app feel less responsive. Severe stalls can cause app termination.

### SwiftUI and UIKit Render Optimization
- **Overdraw**: avoid translucent views layered over other virtual content; GPU must composite all layers even when a fully opaque view above would occlude them — use opaque views wherever possible
- **Window size**: more pixels = more render server work; use `.defaultSize` to keep windows appropriately sized, not larger than needed
- **Dynamic content scaling**: Core Animation layers are rescaled based on gaze position to sharpen text and vector graphics; this triggers redraws even with no app update. SwiftUI/UIKit opt in automatically; custom CA/CG rendering can opt in explicitly.
- **Offscreen render passes**: shadows, blurs, and masks cause expensive offscreen passes — reduce or eliminate these effects
- **Unnecessary view updates**: use `@Observable` (Swift Observation framework) instead of `ObservableObject` for finer-grained change tracking and fewer layout passes

### RealityKit Asset Optimization
- Reality Composer Pro statistics panel shows geometry and draw call counts — lower is better
- **Mesh geometry**: minimize triangle/vertex counts; combine mesh parts that share a material to reduce draw calls
- **Transparency/overdraw**: use transparency sparingly in 3D scenes; prefer "Physically Based" material (environment-lit, well-optimized) for opaque content; use "Custom" material with unlit surface for large/transparent meshes (cheaper — no lighting calculation)
- **Baked lighting**: use lightmap textures or time-based animations instead of real-time dynamic lighting for immersive experiences
- Exported Reality Composer Pro files (`.usdc`, `.usdz`) are pre-optimized for loading time, memory cost, and include automatic texture compression

### RealityKit Runtime Optimization
- **Avoid rapid entity creation/destruction**: create entities in advance; hide/show them with `.isEnabled` flag or by adding/removing from the scene hierarchy
- **Flatten entity hierarchies**: fewer entities updated per frame = less render server work
- **Animation update rates**: for code-driven animations, lower update frequencies and reduce the count of simultaneously animated entities
- **Attachment optimization**: SwiftUI attachments inside `RealityView` obey the same SwiftUI rendering rules — apply all SwiftUI optimization principles to them
- **Async asset loading**: use asynchronous loading APIs; load assets before they are needed; share the same asset across multiple entities using one load

### Metal and CompositorServices Optimization
- Use `CompositorServices` to bypass the render server and submit a rendered surface directly to the compositor
- **Frame pacing**: submit one frame per compositor update; missing a submission causes termination
- **Prediction**: query a new foviation map and post-prediction pose each frame; query input data at the last moment before encoding GPU work — ensures responsive virtual content relative to head/eye motion
- **Shader optimization**: reduce ALU instructions and texture accesses; use compute shaders instead of fragment shaders where possible
- Profile GPU with the Metal System Trace Instruments template; look for long-running vertex and fragment shader times

### User Input Performance
- Input processing happens on the main thread; stalls make the app feel unresponsive
- Target: **< 8 ms** for input update processing at 90 Hz refresh rate
- **RealityKit colliders**: prefer static colliders over dynamic colliders for interactive 3D content (static are cheaper for hit testing)
- Minimize overlapping interactive content to reduce redundant hit-testing work

### ARKit Optimization
- ARKit algorithms run system-wide at all times; every anchor your app adds increases system-wide workload
- **AnchorComponent tracking mode**: use `.once` tracking mode (`AnchorComponent(target:trackingMode: .once)`) instead of continuous tracking when the anchor position does not need to update after initial placement
- Minimize total persistent and transient anchors
- Query ARKit data right before it is applied to content (stale data = visible lag between physics and visuals)
- Post-prediction data is expensive; Metal-based custom engines need it, but RealityKit-based apps do not
- Disable collision generation for scene understanding meshes when not actively needed

### Audio and Video Performance
- Spatial audio processing accounts for user position, surroundings, and source positions in real time — expensive
- To reduce spatial audio load: reduce the number of concurrently playing audio sources, minimize moving sources, and keep the soundstage size appropriate
- Video: each playing video is decoded and rendered by the render server; minimize UI/3D updates during video playback to give the render server headroom
- Prefer 24 or 30 Hz video for optimal GPU/power efficiency over 60 Hz
- Limit the number of concurrent videos playing simultaneously

### SharePlay Performance
- Profile and optimize local performance first; SharePlay syncs render updates across devices — expensive updates multiply across the network
- Disable non-essential app features during SharePlay sessions
- Profile for power during SharePlay to prevent thermal pressure from limiting sustained performance

### Thermal and Memory Pressure
- Subscribe to `ProcessInfo.thermalStateDidChangeNotification` and reduce app workload when thermal state rises
- Use Xcode thermal inducers to simulate elevated thermal states during development
- Terminate risk: apps consuming too much memory or missing render deadlines under thermal pressure are terminated

**Memory reduction priorities for visionOS:**
- UI rendering: minimize offscreen render passes, window count, media content
- RealityKit: reduce texture resolution and mesh/particle geometry sizes
- Audio/video: evaluate memory costs when adjusting resolution, bitrate, format, and duration

**Field data collection:**
- `MetricKit` – diagnostic reports (hangs, CPU time, memory, power) from user devices
- Xcode Organizer – aggregated performance data including energy diagnostics from consenting users

## APIs & Frameworks

- **RealityKit Trace** (Instruments template) **[NEW]** – primary profiling tool for visionOS render performance and power; see "Meet RealityKit Trace" (session 10099)
- **Metal System Trace** (Instruments template) – GPU profiling for Metal and custom material shaders
- **CompositorServices** **[NEW]** – framework for Metal apps to submit frames directly to the compositor, bypassing the render server
- `AnchorComponent(target:trackingMode: .once)` **[NEW]** – single-placement anchor tracking (avoids continuous tracking cost)
- `@Observable` (Swift Observation) **[NEW]** – granular change tracking; reduces unnecessary SwiftUI view updates
- `isEnabled` (RealityKit `Entity`) – hide/show entities without removing from hierarchy (cheaper than add/remove)
- `ProcessInfo.thermalStateDidChangeNotification` – subscribe to thermal state changes for adaptive performance
- `ProcessInfo.thermalState` – `.nominal`, `.fair`, `.serious`, `.critical` — adapt workload accordingly
- **MetricKit** – `MXMetricManager`, `MXDiagnosticReport` — field performance and power diagnostics
- Xcode Organizer – aggregated energy and performance reports from consenting users
- Xcode Thermal Inducer – simulate elevated thermal states in the simulator/device during development
- Reality Composer Pro Statistics panel – scene geometry and draw call counts for asset optimization
- `RealityView` (SwiftUI) **[NEW]** – SwiftUI view hosting RealityKit content; attachments inherit SwiftUI rendering costs
- "Physically Based" material (Reality Composer Pro) – environment-lit; best for opaque meshes
- "Custom" material with unlit surface (Reality Composer Pro) – optimal for transparent/large mesh content; no lighting calculation overhead

## Code Highlights

No code samples were included in this session (it is a strategy/guidance session). For profiling implementation, refer to:
- **"Meet RealityKit Trace"** (session 10099) – RealityKit Trace Instruments walkthrough
- **"Explore rendering for spatial computing"** (session 10095) – dynamic content scaling details
- **"Discover Metal for immersive apps"** – CompositorServices frame submission pattern

Thermal state adaptation pattern:
```swift
NotificationCenter.default.addObserver(
    forName: ProcessInfo.thermalStateDidChangeNotification,
    object: nil, queue: .main
) { _ in
    switch ProcessInfo.processInfo.thermalState {
    case .nominal, .fair:
        enableFullQuality()
    case .serious:
        reduceFidelity()
    case .critical:
        minimizeWork()
    @unknown default:
        break
    }
}
```

## Takeaways
- visionOS renders at 90+ Hz continuously regardless of app updates — every optimization reduces real system load, not just perceived smoothness; inefficient apps directly degrade the overall spatial computing experience
- The two highest-leverage optimizations are: (1) eliminating offscreen render passes (shadows, blurs, masks) from SwiftUI/UIKit content, and (2) using `.once` ARKit tracking mode for anchors that do not need continuous position updates
- Profile on hardware early and often using RealityKit Trace; the simulator does not reproduce the render server workload or thermal behavior of a real device
- Subscribe to `thermalStateDidChangeNotification` and reduce workload gracefully as thermal pressure rises — failing to do so risks app termination under sustained use

---
_Source: WWDC23 Session 10100 page (abstract, chapter summaries, and transcript)._
