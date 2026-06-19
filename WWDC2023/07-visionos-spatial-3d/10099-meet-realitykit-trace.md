# Meet RealityKit Trace
**WWDC23 · Session 10099** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10099/)

_Platforms:_ visionOS 1

## Overview
RealityKit Trace is a new Instruments template introduced in Instruments 15 specifically for profiling spatial computing apps on visionOS. It surfaces RealityKit-specific rendering metrics alongside familiar CPU/GPU and Core Animation data, enabling developers to identify and resolve performance bottlenecks unique to the visionOS rendering pipeline.

The session demonstrates three distinct optimization workflows using the Hello World sample app: reducing offscreen render passes caused by SwiftUI shadows, lowering triangle and vertex counts in 3D assets, and reducing CPU usage and system power impact by avoiding expensive model loading inside SwiftUI view bodies.

Because visionOS renders continuously to account for head movements, even static SwiftUI UI must sustain 90 FPS, making performance optimization more critical than on other Apple platforms.

## Key Topics

### visionOS Rendering Architecture
- Three components: app process, render server, compositor
- **Shared Space**: multiple apps render in the same render server; neighboring apps affect your app's performance budget
- **Full Space**: all other apps hidden; full performance budget available to your app
- Profiling recommendation: profile in isolation to measure your app's own impact; profile alongside other apps to simulate real user scenarios
- Target frame rate: 90 FPS (OS may target lower based on content and environment)

### RealityKit Trace Template (Instruments 15)
- Available for real devices and simulator; real device recommended for accurate timing
- Simulator useful for quick iteration on non-time-based statistics

### RealityKit Frames Instrument
- Tracks every rendered frame with color-coded status: green (well within deadline), orange (just within deadline), red (exceeded deadline / dropped)
- Shows average CPU and GPU time per frame
- Enables zooming in to inspect individual frame stage durations

### RealityKit Metrics Instrument
- Detects and surfaces bottlenecks across the entire render pipeline
- Detail view: bottleneck summary by severity and type
- Extended detail view: specific bottleneck description + recommendations
- Expanded tracks: Core Animation, 3D Render, System Power Impact, RealityKit Systems timing
- Key metric thresholds highlighted visually (color coded) to indicate acceptable vs. problematic values
- System Power Impact lane: nominal, elevated, high — apps should stay nominal as much as possible

### Optimizing Offscreen Passes (Core Animation)
- Core Animation statistics track: offscreen prepares, render passes, transparency/blur
- Offscreen passes require the renderer to pause main rendering and complete a side task; particularly expensive on visionOS due to continuous rendering
- Main causes: shadows, masking, rounded rectangles, visual effects
- Optimization: remove `.shadow(radius:)` modifiers from SwiftUI views that don't need them
- Result in demo: 4× reduction in offscreen passes

### Optimizing Asset Rendering (3D Render Track)
- 3D Render metrics: triangle count, vertex count, draw calls — each has recommended thresholds
- Optimization: use simpler meshes; verify complexity in Reality Composer Pro Statistics panel
- Take advantage of instancing for identical meshes
- Result in demo: substantially reduced triangle/vertex counts with no perceptible quality loss

### Optimizing System Power Impact
- High CPU + GPU usage drives power into elevated/high states
- Anti-pattern: calling expensive operations (model loading, collision shape generation) inside SwiftUI view bodies — view body recomputes on every state change
- Fix: instantiate `Entity` objects in `ObservableObject` ViewModel as `@Published` properties; reuse them from the view body
- Time Profiler identifies heaviest stack traces; use "Extended Detail" view for call tree analysis
- Result in demo: CPU usage dropped from 100% to ~10%; power impact returned to nominal

### Other Profiling Tools for Spatial Apps
- **SwiftUI Instruments** – SwiftUI-specific performance analysis
- **Core Animation Instrument** – offscreen passes, render passes, transparency
- **Hangs Instrument** – hang detection
- **Time Profiler** – CPU-bound code analysis; heavy stack trace view
- **Metal System Trace template** – GPU timeline, GPU counters, GPU performance states for Metal content
- **Reality Composer Pro Statistics panel** – triangle/vertex counts per asset during content authoring

## APIs & Frameworks

- **RealityKit Trace** Instruments template **[NEW]** – visionOS performance profiling
- **RealityKit Frames** instrument **[NEW]** – per-frame render timing and deadline classification
- **RealityKit Metrics** instrument **[NEW]** – bottleneck detection, Core Animation stats, 3D Render stats, System Power Impact, RealityKit Systems timing
- Core Animation metrics: offscreen prepares count, render pass count, transparency/blur usage
- 3D Render metrics: triangle count, vertex count, draw call count (with recommended thresholds)
- System Power Impact lane: nominal / elevated / high states
- RealityKit Systems timing: built-in + custom `System` execution time per frame
- **Instruments 15** – host for RealityKit Trace
- `Entity.makeModel(name:filename:radius:color:)` – entity factory (anti-pattern: do not call in SwiftUI view body)
- `Entity.generateCollisionShapes(recursive:)` – expensive; pre-generate and cache
- `ObservableObject` / `@Published` – correct pattern for caching pre-built entities in a ViewModel
- **SwiftUI `.shadow(radius:)`** modifier – use sparingly on visionOS (causes offscreen passes)
- **Shared Space** / **Full Space** – visionOS rendering environments with different performance implications
- **Metal System Trace** Instruments template – GPU profiling for Metal-based content

## Code Highlights

Anti-pattern — expensive model loading in SwiftUI view body:
```swift
struct Orbit: View {
    var body: some View {
        Earth(world: EarthEntity.makeGlobe(), ...) // Regenerates entity on every view update
    }
}
```

Fix — cache entity in ViewModel:
```swift
class ViewModel: ObservableObject {
    @Published var orbitEarthEntity: EarthEntity = .makeGlobe() // Created once
}

struct Orbit: View {
    @EnvironmentObject private var model: ViewModel
    var body: some View {
        Earth(world: model.orbitEarthEntity, ...) // Reuses cached entity
    }
}
```

SwiftUI shadow causing offscreen passes (use sparingly):
```swift
.shadow(radius: 10) // Triggers offscreen render pass on visionOS
```

## Takeaways
- visionOS requires continuous rendering at up to 90 FPS even for static UI; every Shadow, mask, and visual effect in SwiftUI incurs offscreen render passes that directly impact frame deadlines.
- Never load models or generate collision shapes inside a SwiftUI view body — cache them in an `ObservableObject` ViewModel and reference them from the body.
- Use the 3D Render track's triangle/vertex thresholds as guardrails during asset creation; Reality Composer Pro's Statistics panel is the earliest opportunity to catch over-complex geometry.
- Profile in isolation to measure your app's own system power impact; only profile alongside other apps to simulate real user experience in Shared Space.

---
_Source: WWDC23 Session 10099 page (abstract, chapter summaries, code samples, and resource links)._
