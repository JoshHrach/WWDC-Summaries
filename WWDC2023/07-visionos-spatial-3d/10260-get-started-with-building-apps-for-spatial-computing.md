# Get Started with Building Apps for Spatial Computing
**WWDC23 · Session 10260** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10260/)

_Platforms:_ visionOS 1

## Overview
This introductory session establishes the foundational vocabulary and architecture for building apps on Apple's new spatial computing platform (visionOS). It covers the three fundamental building blocks — windows, volumes, and spaces — and how they combine to deliver experiences ranging from familiar 2D UI to fully immersive spatial environments.

The session also walks through the developer tooling ecosystem: Xcode 15's 3D SwiftUI Previews, the visionOS Simulator with simulated gestures, Instruments with the new RealityKit Trace template, and Reality Composer Pro for authoring and previewing 3D content. A hands-on code walkthrough using the "Hello World" sample app demonstrates how to use `WindowGroup`, `Model3D`, `RealityView`, and attachments in a real visionOS project.

Privacy is highlighted as a first-class platform principle: the system manages sensor data and delivers only high-level events (touch, hover effects) to apps, with explicit permission required for more sensitive capabilities like hand tracking and scene understanding.

## Key Topics

### Windows, Volumes, and Spaces
- **Windows**: SwiftUI scenes containing traditional 2D views and controls, optionally mixed with 3D content. Multiple windows can coexist in the Shared Space.
- **Volumes**: A new `windowStyle(.volumetric)` scene type for displaying bounded 3D content. Coexists with other apps in the Shared Space; content must remain within declared bounds. Specified with `defaultSize(width:height:depth:)`.
- **Shared Space**: Default app environment; multiple apps coexist, people see passthrough.
- **Full Space (Immersive Space)**: App takes exclusive view. Supports `mixed` (passthrough overlay), `full` (fully immersive, no passthrough), and `progressive` (Digital Crown adjustable) immersion styles.

### Input and Interaction
- Eye and hand-based interaction: looking at an element and pinching fingers selects it; reaching out and touching works for nearby content.
- System detects taps, long presses, drags, rotations, zooms — delivered as standard touch events.
- SwiftUI gesture APIs work seamlessly with RealityKit entities.
- ARKit Skeletal Hand Tracking for custom hand-based interactions in Full Space.
- Wireless keyboards, trackpads, accessibility hardware, and Game Controllers are automatically supported.

### SharePlay and Shared Context
- `SharePlay` / `Group Activities` framework supports shared windows; system syncs orientation, scale, animations.
- Shared context ensures all participants in a SharePlay session experience content the same way.
- Spatial Persona Templates customize collaborative experiences.

### Developer Tools
- **Xcode 15**: 3D SwiftUI Preview Canvas for visualizing RealityKit scenes including animations; object mode for 3D layout previewing.
- **visionOS Simulator**: Keyboard/mouse/game controller navigation; simulated system gestures; three built-in environments with day/night lighting.
- **Instruments 15**: New RealityKit Trace template — GPU/CPU/power analysis, frame bottleneck identification, entity count metrics.
- **Reality Composer Pro**: Preview and prepare 3D assets; author MaterialX custom materials; particle effects; spatial audio preview.

### App Migration Path
- iPad and iPhone apps run natively in visionOS Shared Space (iPad variant preferred).
- Adding a visionOS destination in Xcode: one click, recompile for native spacing, sizing, relayout, and platform materials.
- Existing controls automatically gain hover highlighting.

## APIs & Frameworks
- `SwiftUI` — primary UI framework for visionOS **[NEW visionOS support]**
- `WindowGroup` — scene type for windows **[NEW visionOS support]**
- `windowStyle(.volumetric)` **[NEW]** — creates a volume-style 3D scene
- `defaultSize(width:height:depth:)` **[NEW]** — specifies volume dimensions in points or meters
- `ImmersiveSpace` **[NEW]** — scene type for Full Space experiences
- `ImmersionStyle.mixed` **[NEW]** — layers content over passthrough
- `ImmersionStyle.full` **[NEW]** — replaces passthrough with fully rendered content
- `ImmersionStyle.progressive` **[NEW]** — Digital Crown-adjustable immersion level
- `Model3D` **[NEW]** — SwiftUI view for loading and displaying USDZ/3D models via RealityKit
- `RealityView` **[NEW]** — SwiftUI view hosting RealityKit entities with make/update/attachments closures
- `RealityView` attachments **[NEW]** — positions SwiftUI views within a 3D scene via tag-based entity binding
- `DragGesture(targetedToEntity:)` **[NEW]** — drag gesture targeting a specific RealityKit entity
- `TapGesture` (3D) **[NEW]** — tap gesture for 3D objects
- `onHover` modifier — hover detection for SwiftUI views on visionOS
- `RotateGesture3D` **[NEW]** — 3D rotation gesture
- `RealityKit` — 3D rendering and ECS framework for visionOS **[NEW visionOS support]**
- `ARKit` — scene understanding, plane estimation, Skeletal Hand Tracking **[NEW visionOS support]**
- ARKit Skeletal Hand Tracking **[NEW]** — detailed per-joint hand pose data in Full Space
- `Group Activities` / `SharePlay` — multi-user collaborative experiences **[NEW visionOS support]**
- Spatial Persona Templates **[NEW]** — customization of collaborative spatial presence
- Reality Composer Pro **[NEW]** — macOS app for authoring visionOS 3D content
- RealityKit Trace (Instruments 15 template) **[NEW]** — performance profiling for spatial apps
- `Game Controller` framework — wireless controller input **[NEW visionOS support]**

## Code Highlights

Creating a volumetric scene:
```swift
WindowGroup {
    ContentView()
}
.windowStyle(.volumetric)
.defaultSize(width: 0.5, height: 0.5, depth: 0.5, in: .meters)
```

Embedding a 3D model in a SwiftUI view and enabling drag:
```swift
Model3D(named: "Satellite") { model in
    model.resizable()
} placeholder: {
    ProgressView()
}
.gesture(
    DragGesture()
        .targetedToEntity(satelliteEntity)
        .onChanged { value in
            // move satellite with drag value
        }
)
```

Using RealityView with attachments:
```swift
RealityView { content, attachments in
    // make: add entities to content
} update: { content, attachments in
    if let pin = attachments.entity(for: "pin") {
        content.add(pin)
    }
} attachments: {
    Attachment(id: "pin") {
        Image("bakery-icon")
    }
}
```

Adding an immersive space with full immersion:
```swift
ImmersiveSpace(id: "solar-system") {
    RealityView { content in
        // add Earth entity
    }
}
.immersionStyle(selection: $immersionStyle, in: .full)
```

## Takeaways
- The Windows → Volumes → Spaces progression is a spectrum of immersion; well-designed visionOS apps flex between these elements based on what the moment requires.
- `Model3D` and `RealityView` (with attachments) are the two primary entry points for integrating 3D content into SwiftUI-based visionOS apps.
- Privacy is foundational: apps receive abstracted events (touch, hover) rather than raw sensor data; sensitive capabilities (hand tracking, scene understanding) require explicit user permission.
- Existing iPad apps run immediately on visionOS with a single Xcode destination addition, providing a low-friction migration path before adding visionOS-specific spatial features.

---
_Source: WWDC23 Session 10260 page (abstract, chapter summaries, code samples, and resource links)._
