# Develop Your First Immersive App
**WWDC23 · Session 10203** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10203/)

_Platforms:_ visionOS 1

## Overview
This end-to-end tutorial session by Peter from the RealityKit Tools team walks developers through creating a visionOS app from scratch using Xcode, the visionOS Simulator, Reality Composer Pro, and RealityKit. It covers every major building block: the new Xcode project assistant with visionOS-specific scene types, the `RealityView` SwiftUI view, loading USDZ content from a RealityKit content package, opening an `ImmersiveSpace`, and targeting gestures to specific RealityKit entities.

The session is the canonical "getting started" guide for visionOS development and maps directly to the engineering companion sessions on SwiftUI, RealityKit, and Reality Composer Pro.

## Key Topics

### Xcode Project Setup for visionOS
The new project assistant has two new options for visionOS:
- **Initial Scene**: Choose between `Window` (2D-primary content, resizable in XY, shown alongside other apps) or `Volume` (3D-primary content, app-controlled size in all three dimensions, shown alongside other apps).
- **Immersive Space**: Optional; adds a second scene that hides other apps and enters a Full Space. Three styles: `mixed` (passthrough remains), `progressive` (adjustable portal via Digital Crown), `full` (passthrough hidden, full environment replacement).

Best practice: always start in a window/volume; provide explicit controls to enter and exit immersive experiences.

### RealityView **[NEW]**
`RealityView` is the new SwiftUI view type for embedding RealityKit content in a SwiftUI hierarchy. It takes a `make` closure (initial content setup, runs once, async) and an optional `update` closure (runs when SwiftUI state changes — not every frame). Content is added via `content.add(_:)`. A `TapGesture` or other SwiftUI gesture can be attached directly to the `RealityView`.

### Simulator for visionOS **[NEW]**
The Simulator presents a simulated spatial environment. Navigation controls let you look around, pan, orbit, and move. Multiple simulated scenes (rooms, lighting conditions) are available from the toolbar menu. Interaction: pointer controls gaze; click = tap; click-hold = pinch.

### Xcode Previews for visionOS
Standard Xcode Previews work with visionOS. For content that extends beyond default scene bounds (immersive content), apply `.previewLayout(.sizeThatFits)` to the preview view.

### Reality Composer Pro **[NEW]**
A standalone tool for preparing, authoring, and previewing RealityKit content packages. Content packages are Swift packages containing USDZ assets and scene graphs; they are processed at build time. Scenes can be created and named, assets imported via drag-and-drop, transforms set numerically in the Inspector. Use the "Open in Reality Composer Pro" button from Xcode's package editor.

**Coordinate system for ImmersiveSpace**: Origin is inferred foot position. +X = right, +Y = up, −Z = forward.

### Opening an ImmersiveSpace **[NEW]**
Declare an `ImmersiveSpace` scene in the app's `body` with a string ID. Use `@Environment(\.openImmersiveSpace)` to capture the environment action, then call it asynchronously in a `Task` from a button.

### Entity Targeting **[NEW]**
The `.targetedToAnyEntity()` modifier on a gesture attached to a `RealityView` identifies which specific `Entity` within the view was interacted with. Requirements: the entity must have both a `CollisionComponent` and an `InputTargetComponent`. Both can be added in Reality Composer Pro or programmatically. The gesture's `onEnded` closure receives a value with an `.entity` property.

### RealityKit Animations
`entity.move(to:relativeTo:duration:timingFunction:)` animates an entity's transform. Timing functions include `.easeInOut`.

## APIs & Frameworks

### SwiftUI — visionOS Scene Types **[NEW]**
- `WindowGroup` — declares a window or volume scene
- `.windowStyle(.volumetric)` — makes a WindowGroup present as a 3D volume **[NEW]**
- `ImmersiveSpace(id:)` — declares an immersive space scene **[NEW]**
- `.immersionStyle(selection:in:)` — sets the immersion style (`.mixed`, `.progressive`, `.full`) **[NEW]**
- `@Environment(\.openImmersiveSpace)` — environment action to open an immersive space **[NEW]**
- `@Environment(\.dismissImmersiveSpace)` — environment action to close an immersive space **[NEW]**
- `.previewLayout(.sizeThatFits)` — allows Xcode Previews to show unbounded immersive content **[NEW]**
- `.glassBackgroundEffect()` — applies the visionOS glass material to a SwiftUI container **[NEW]**

### RealityKit **[NEW/Updated]**
- `RealityView` — SwiftUI view for embedding RealityKit content **[NEW]**
- `RealityView.make` closure — async, adds initial entities via `content.add(_:)`
- `RealityView.update` closure — called on SwiftUI state change
- `Entity(named:in:)` — async loader for a named scene from a RealityKit content bundle **[NEW]**
- `Entity.transform` — the entity's `Transform` (position, rotation, scale as `SIMD3<Float>`)
- `Entity.move(to:relativeTo:duration:timingFunction:)` — plays a transform animation **[NEW]**
- `CollisionComponent` — required on an entity for gesture targeting **[NEW]**
- `InputTargetComponent` — required on an entity for gesture targeting **[NEW]**
- `TapGesture().targetedToAnyEntity()` — modifier targeting any entity within a RealityView **[NEW]**
- `.targetedToEntity(_:)` — targets a specific entity **[NEW]**
- `RealityKitContent` module — Swift package generated from a Reality Composer Pro content package **[NEW]**
- `realityKitContentBundle` — the Bundle for the RealityKit content package

### Reality Composer Pro **[NEW]**
- New standalone tool for scene authoring, USDZ import, collision shape authoring, component assignment
- Scenes exported as part of a Swift package (RealityKit content package)
- `CollisionComponent` and `InputTargetComponent` added via Reality Composer Pro Inspector

## Code Highlights

```swift
// Glass-effect SwiftUI button container
VStack {
    Toggle("Enlarge RealityView Content", isOn: $enlarge)
        .toggleStyle(.button)
}
.padding()
.glassBackgroundEffect()
```

```swift
// RealityView with make and update closures
RealityView { content in
    if let scene = try? await Entity(named: "Scene", in: realityKitContentBundle) {
        content.add(scene)
    }
} update: { content in
    if let scene = content.entities.first {
        let uniformScale: Float = enlarge ? 1.4 : 1.0
        scene.transform.scale = [uniformScale, uniformScale, uniformScale]
    }
}
.gesture(TapGesture().targetedToAnyEntity().onEnded { _ in
    enlarge.toggle()
})
```

```swift
// Open an ImmersiveSpace from a button
@Environment(\.openImmersiveSpace) var openImmersiveSpace

Button("Open") {
    Task { await openImmersiveSpace(id: "ImmersiveSpace") }
}
```

```swift
// Animate entity on tap using entity targeting
.gesture(TapGesture().targetedToAnyEntity().onEnded { value in
    var transform = value.entity.transform
    transform.translation += SIMD3(0.1, 0, -0.1)
    value.entity.move(to: transform, relativeTo: nil,
                      duration: 3, timingFunction: .easeInOut)
})
```

## Takeaways
- `RealityView` is the fundamental bridge between SwiftUI and RealityKit on visionOS; use the `make` closure for one-time async loading and `update` for SwiftUI state-driven changes.
- Always start apps in a window or volume; require explicit user action (a button) to enter an `ImmersiveSpace`.
- For entity targeting, every interactive RealityKit entity must have both a `CollisionComponent` and an `InputTargetComponent` — add these in Reality Composer Pro or in code.
- Use Reality Composer Pro to author scenes visually; changes are reflected immediately in Xcode Previews via the RealityKit content package.

---
_Source: WWDC23 Session 10203 page (abstract, chapter summaries, code samples, and resource links)._
