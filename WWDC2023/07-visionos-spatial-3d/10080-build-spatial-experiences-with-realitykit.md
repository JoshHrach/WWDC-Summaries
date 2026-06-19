# Build Spatial Experiences with RealityKit
**WWDC23 · Session 10080** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10080/)

_Platforms:_ visionOS 1

## Overview
This session introduces RealityKit as the core 3D framework for visionOS, covering how to bring 3D models and spatial experiences into apps using the new RealityView SwiftUI view. Using Apple's "Hello World" sample app as the running example, the session walks through progressively richer ways to display 3D content: embedding a model in a 2D window, presenting it in a volumetric window, and finally placing it inside a fully immersive space.

The session then dives into the fundamental building blocks of RealityKit—entities, components, and systems (ECS)—explaining how predefined components like `ModelComponent`, `TransformComponent`, `CollisionComponent`, and `InputTargetComponent` are composed on entities, and how developers can define their own custom components and systems to implement unique behaviors.

The final third covers interactivity (drag gestures with `targetedToEntity`), the three types of spatial audio, animation types (from-to-by, orbit, time-sampled), and how to structure app-wide behavior logic using the `System` protocol with `EntityQuery` filters.

## Key Topics

### RealityKit + SwiftUI Integration
- `Model3D` for simple async loading of USD/USDZ models inside any SwiftUI view.
- Volumetric window style (`WindowGroup` + `.windowStyle(.volumetric)`) with real-world meter sizing.
- `ImmersiveSpace` scene type for content that escapes window bounds.
- `RealityView` as the primary API for composing multiple entities with full control.

### Entities and Components
Entities are containers; components supply behavior. Core components: `ModelComponent` (mesh + materials), `TransformComponent` (position/orientation/scale in meter-based coordinates), `CollisionComponent`, `InputTargetComponent`, `HoverEffectComponent`, `SpatialAudioComponent`, `AmbientAudioComponent`, `ChannelAudioComponent`.

### RealityView API
The `make` closure is async, enabling `async let` entity loading. An `update` closure re-runs on state changes, connecting SwiftUI data to RealityKit component properties. Coordinate conversion between SwiftUI and RealityKit spaces is done via `content.convert(_:from:to:)`.

### Input and Gestures
Entities must have both `CollisionComponent` and `InputTargetComponent` to receive gestures. SwiftUI gestures gain a `.targetedToEntity(_:)` modifier that routes them to specific entities and provides 3D location values that must be coordinate-converted with `value.convert(_:from:to:)`.

### Animation
`OrbitAnimation`, from-to-by animations, and time-sampled animations are built-in. Resources are generated with `AnimationResource.generate(with:)` and played with `entity.playAnimation(_:)`. `AnimationEvents.PlaybackCompleted` can be observed via `content.subscribe(to:)`.

### Spatial Audio
Three components: `SpatialAudioComponent` (positional, with `directivity` beam control), `AmbientAudioComponent` (multichannel environment sound), `ChannelAudioComponent` (direct-to-speaker for background music). Audio resources loaded with `AudioFileResource(named:configuration:)` and played with `entity.playAudio(_:)`.

### Custom Components and Systems
Custom components conform to `Component` (and optionally `Codable` for Reality Composer Pro integration). Systems conform to `System`, define an `EntityQuery`, and implement `update(context:)` at rendering cadence. Systems are registered once globally via `MySystem.registerSystem()`.

## APIs & Frameworks

**RealityKit** (all new on visionOS 1 unless noted)
- `RealityView` **[NEW]** — SwiftUI view hosting RealityKit entities
- `RealityView.make` closure with `RealityViewContent` — async entity loading
- `RealityView.update` closure — state-driven entity updates
- `RealityViewContent.add(_:)` — add entity to scene
- `RealityViewContent.convert(_:from:to:)` — coordinate space conversion
- `RealityViewContent.subscribe(to:)` — RealityKit event subscription
- `Model3D(named:content:placeholder:)` **[NEW]** — simple async 3D model loader
- `ModelEntity(named:)` async — load entity from USD file
- `Entity` — base container class
- `ModelComponent` — mesh and materials
- `TransformComponent` — 3D position, orientation, scale
- `CollisionComponent` — required for input hit-testing
- `InputTargetComponent` **[NEW]** — marks entity as gesture target
- `HoverEffectComponent` **[NEW]** — system-rendered hover highlight (privacy-preserving)
- `SpatialAudioComponent(directivity:)` **[NEW]** — positional audio with beam directivity
- `AmbientAudioComponent` **[NEW]** — multichannel environment audio
- `ChannelAudioComponent` **[NEW]** — direct speaker output audio
- `AudioFileResource(named:configuration:)` async — load audio file
- `Entity.playAudio(_:)` — play an audio resource on an entity
- `Entity.playAnimation(_:)` — play an animation resource
- `Entity.availableAnimations` — animations authored in USD
- `AnimationResource.generate(with:)` — generate from animation definition
- `OrbitAnimation` **[NEW]** — revolve entity around parent
- `AnimationEvents.PlaybackCompleted` — animation completion event
- `EntityQuery(where:)` — filter entities by component presence
- `System` protocol **[NEW]** — app-wide update logic
- `SceneUpdateContext` — passed to System.update; provides entity iteration
- `Component` protocol — custom component conformance
- `EventSubscription` — handle returned from subscribe
- `GeometryReader3D` **[NEW]** — 3D equivalent of GeometryReader

**SwiftUI (visionOS extensions)**
- `WindowGroup.windowStyle(.volumetric)` **[NEW]** — volumetric window
- `WindowGroup.defaultSize(width:height:depth:in:.meters)` **[NEW]** — real-world sizing
- `ImmersiveSpace(id:)` **[NEW]** — immersive scene type
- `openImmersiveSpace` environment action **[NEW]**
- `openWindow` environment action
- `DragGesture().targetedToEntity(_:)` **[NEW]** — entity-targeted gesture modifier

## Code Highlights

Simple Model3D with placeholder:
```swift
Model3D(named: "Globe") { model in
    model.resizable().scaledToFit()
} placeholder: {
    ProgressView()
}
```

Async entity loading with positioning in RealityView:
```swift
RealityView { content in
    async let earth = ModelEntity(named: "Earth")
    async let moon = ModelEntity(named: "Moon")
    if let earth = try? await earth, let moon = try? await moon {
        content.add(earth)
        content.add(moon)
        moon.position = [0.5, 0, 0]
    }
}
```

Orbit animation:
```swift
let orbit = OrbitAnimation(name: "Orbit", duration: 30, axis: [0, 1, 0],
                           startTransform: moon.transform,
                           bindTarget: .transform, repeatMode: .repeat)
if let animation = try? AnimationResource.generate(with: orbit) {
    moon.playAnimation(animation)
}
```

## Takeaways
- `RealityView` is the primary API for composing multi-entity 3D scenes on visionOS, with async loading, state-driven updates, and coordinate conversion built in.
- Entities need both `CollisionComponent` and `InputTargetComponent` to receive gestures; `HoverEffectComponent` provides privacy-preserving visual feedback.
- The ECS (Entity-Component-System) pattern lets developers encapsulate custom data in `Component`-conforming types and implement global update logic in `System`-conforming types registered once at app launch.
- Spatial Audio on visionOS comes in three flavors (Spatial, Ambient, Channel), each suited to different content types and configured with dedicated RealityKit components.

---
_Source: WWDC23 Session 10080 page (abstract, chapter summaries, code samples, and resource links)._
