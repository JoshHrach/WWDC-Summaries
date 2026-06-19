# Better Together: SwiftUI and RealityKit
**WWDC25 · Session 274** · [Watch](https://developer.apple.com/videos/play/wwdc2025/274/)

_Platforms:_ visionOS 26

## Overview
This session explores the deepening integration between SwiftUI and RealityKit in visionOS 26, demonstrating how to blend 2D UI and interactive 3D content within the same app. Using a pair of robots as narrative anchors, the session walks through enhancements that let each framework feed data and behavior to the other while minimizing boilerplate.

Key themes are: Model3D gaining animation and configuration capabilities, a smooth upgrade path from Model3D to RealityView, a new Object Manipulation API for hand-based interaction, new RealityKit component types that embed SwiftUI views and gestures directly onto entities, bidirectional observable data flow between the two frameworks, unified 3D coordinate conversion, and SwiftUI-driven implicit animation for RealityKit components.

## Key Topics

### Model3D Enhancements
`Model3DAsset` is a new type that loads a 3D asset asynchronously and exposes its animations. The asset vends an `animationPlaybackController` (now Observable) allowing SwiftUI views to pause, resume, and scrub animations via a Slider. `ConfigurationCatalog` support lets Model3D switch between different mesh or material representations of an entity at runtime — useful for customizable characters and product configurators.

### Transitioning from Model3D to RealityView
The new `.realityViewLayoutBehavior(_:)` modifier (`.fixedSize`, `.centered`, `.flexible`) lets a RealityView tightly wrap entity bounds, matching the size behavior of Model3D. This enables smooth replacement without layout regressions.

### Object Manipulation API
The `.manipulable()` SwiftUI modifier enables direct/indirect hand-based translate, rotate, and scale on any View or Model3D. On the RealityKit side, `ManipulationComponent.configureEntity(_:)` adds the needed `CollisionComponent`, `InputTargetComponent`, `HoverEffectComponent`, and `ManipulationComponent` in one call. `ManipulationEvents` (WillBegin, DidUpdateTransform, WillRelease, WillEnd, DidHandOff) let apps respond to interaction.

### New SwiftUI RealityKit Components
Three new components bring SwiftUI capabilities directly onto entities:
- **`ViewAttachmentComponent`** — attaches any SwiftUI View to an entity's coordinate space.
- **`GestureComponent`** — attaches SwiftUI gestures (e.g., `TapGesture`) to an entity; values reported in entity coordinate space.
- **`PresentationComponent`** — presents SwiftUI presentations (e.g., popovers) anchored to an entity.

### Bidirectional Information Flow
Entities in visionOS 26 are now Observable. Reading `entity.observable` property creates a SwiftUI dependency on `position`, `scale`, `rotation`, `children`, or any `Component`. This lets entity state drive SwiftUI view updates (e.g., a minimap). Infinite loops are avoided by not writing observed state inside the `RealityView` update closure.

### Unified Coordinate Conversion
The `CoordinateSpace3D` protocol (Spatial framework) is now conformed to by `Entity`, `Scene`, and `GeometryProxy3D`. Values can be converted between any two conforming coordinate spaces, bridging SwiftUI and RealityKit measurements including points-to-meters and axis direction.

### SwiftUI-Driven Implicit Animation for RealityKit
`content.animate()` inside a RealityView update closure, or `Entity.animate(_:_:)`, lets developers animate RealityKit component changes using SwiftUI `Animation` values (e.g., `.bouncy`). Supported components: `Transform`, Audio components, `ModelComponent` (color), light components.

## APIs & Frameworks

**RealityKit (visionOS 26)**
- **[NEW]** `Model3DAsset` — async-loadable asset with animation support
- **[NEW]** `AnimationPlaybackController` — now `Observable`; `.time`, `.isPlaying`, `.duration`, `.pause()`, `.resume()`
- **[NEW]** `Model3D(asset:)`, `Model3D(from:configurations:)` initializers
- **[NEW]** `.realityViewLayoutBehavior(_:)` modifier — `.fixedSize`, `.centered`, `.flexible`
- **[NEW]** `.manipulable(operations:inertia:)` SwiftUI modifier
- **[NEW]** `ManipulationComponent` — ECS component for object manipulation
- **[NEW]** `ManipulationComponent.configureEntity(_:hoverEffect:allowedInputTypes:collisionShapes:)`
- **[NEW]** `ManipulationEvents` — `WillBegin`, `DidUpdateTransform`, `WillRelease`, `WillEnd`, `DidHandOff`
- **[NEW]** `ViewAttachmentComponent(rootView:)`
- **[NEW]** `GestureComponent(_:)`
- **[NEW]** `PresentationComponent` with `configuration: .popover`
- **[NEW]** `entity.observable` property — makes entity properties observable
- **[NEW]** `content.animate()` in RealityView update closure
- **[NEW]** `Entity.animate(_:_:)`
- `ParticleEmitterComponent` — particle effects on entities
- `AnimationLibraryComponent` — named animation access
- `CollisionComponent`, `InputTargetComponent`, `HoverEffectComponent`

**Spatial Framework**
- **[NEW]** `CoordinateSpace3D` protocol
- **[NEW]** `GeometryProxy3D.coordinateSpace3D()`
- `onGeometryChange3D` modifier

## Code Highlights
Load and animate a Model3DAsset:
```swift
let asset = try? await Model3DAsset(named: "sparky")
Model3D(asset: asset)
// Access controller
asset.animationPlaybackController?.resume()
```

Enable object manipulation on a RealityKit entity:
```swift
ManipulationComponent.configureEntity(sparky)
```

Attach a SwiftUI view to an entity:
```swift
let attachment = ViewAttachmentComponent(rootView: NameSign("Bolts"))
let nameSign = Entity(components: attachment)
```

SwiftUI-driven implicit animation:
```swift
content.animate { // uses surrounding SwiftUI transaction animation
    sparky.transform.translation = newPosition
}
```

## Takeaways
- Use `Model3DAsset` and `animationPlaybackController` to add scrubable animation playback to Model3D with standard SwiftUI controls.
- Apply `.realityViewLayoutBehavior(.fixedSize)` when migrating from Model3D to RealityView to preserve layout without regressions.
- Adopt `ViewAttachmentComponent`, `GestureComponent`, and `PresentationComponent` to embed SwiftUI directly into your RealityKit scene graph.
- Use `entity.observable` to drive SwiftUI state from RealityKit — but avoid writing observed state in the RealityView `update` closure to prevent infinite loops.

---
_Source: WWDC25 Session 274 page (abstract, chapter summaries, code samples, and resource links)._
