# What's new in RealityKit
**WWDC25 · Session 287** · [Watch](https://developer.apple.com/videos/play/wwdc2025/287/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26

## Overview
RealityKit receives a broad set of new components and APIs in 2025. The headlining addition is **tvOS support** — RealityKit now runs on all Apple TV 4K generations. The session builds a spatial puzzle game to demonstrate five major features: ARKit data access directly through RealityKit via `SpatialTrackingSession` and `AnchorStateEvents`; the new `ManipulationComponent` for grab/rotate/hand-swap interactions; scene understanding flags for real-world collision and physics; `EnvironmentBlendingComponent` for realistic occlusion by static surroundings; and `MeshInstancesComponent` for GPU-efficient multi-instance mesh rendering.

Additional announcements cover `ImagePresentationComponent` (2D, spatial photo, spatial scenes), expanded `VideoPlayerComponent` immersive video format support, spatial accessory tracking, new SwiftUI integration components, entity attachment API, AVIF texture support, `HoverEffectComponent` GroupIDs, and post-processing effects in `RealityView`.

## Key Topics

### ARKit data via RealityKit (AnchorStateEvents)
`SpatialTrackingSession` (with plane tracking) makes `AnchorEntity` more powerful by firing `AnchorStateEvents`. The `DidAnchor` event delivers an `ARKitAnchorComponent` containing the raw ARKit anchor (cast to `PlaneAnchor`, `HandAnchor`, etc.), exposing `originFromAnchorTransform`, `geometry.extent.anchorFromExtentTransform`, and more — without needing a separate ARKit session.

### ManipulationComponent
`ManipulationComponent.configureEntity(_:collisionShapes:)` automatically adds `InputTargetComponent`, `CollisionComponent`, `HoverEffectComponent`, and `ManipulationComponent` to an entity. `releaseBehavior` can be `.animateToStart` (default) or `.stay`. `ManipulationEvents` — `WillBegin`, `WillEnd`, `DidUpdateTransform`, `DidHandOff`, `WillRelease` — enable toggling physics modes when interaction starts/ends.

### Scene understanding flags
`SpatialTrackingSession.Configuration.sceneUnderstanding` accepts `.collision` and `.physics` flags to add the room mesh to the collision/physics simulation on visionOS. Set these alongside tracking flags before running the session.

### EnvironmentBlendingComponent
Entities with `EnvironmentBlendingComponent(preferredBlendingMode: .occluded(by: .surroundings))` are realistically occluded by static real-world geometry. Note: these entities always render behind other virtual objects; dynamic objects (people, pets) do not cause occlusion.

### MeshInstancesComponent
`MeshInstancesComponent` draws one mesh multiple times from a single entity using a `LowLevelInstanceData` object (providing transforms per instance). Only mesh data is sent to the GPU once, improving performance when many duplicates are needed. `meshInstancesComponent[partIndex: 0] = instances`. On non-visionOS platforms, a `LowLevelBuffer` can pass per-instance render data to `CustomMaterial`.

### ImagePresentationComponent (NEW)
Presents three image types:
- 2D image: `ImagePresentationComponent(contentsOf: url)` (async)
- Spatial photo: same init, set `desiredViewingMode` to `.spatialStereo` or `.spatialStereoImmersive`
- Spatial scene: create `Spatial3DImage(contentsOf:)`, then `ImagePresentationComponent(spatial3DImage:)`, call `spatial3DImage.generate()` — `availableViewingModes` updates to include `.spatial3D` / `.spatial3DImmersive`

### VideoPlayerComponent (expanded)
Now supports spatial video (portal and immersive), Apple Projected Media Profile (180°, 360°, wide-FOV), and Apple Immersive Video. Comfort settings are respected automatically.

### Other additions
- **Spatial accessory tracking** (6DoF, haptics) — watch "Explore spatial accessory input on visionOS"
- **`ViewAttachmentComponent`**, **`PresentationComponent`**, **`GestureComponent`** — SwiftUI integration
- **Entity `.attach(to: pin:)`** — attach meshes to animated skeleton joints
- **`Entity(from: Data)`** — load entities from in-memory Data (e.g., from network)
- **AVIF texture support** — comparable quality to JPEG, 10-bit color, significantly smaller
- **`HoverEffectComponent` GroupIDs** — share hover activation between non-hierarchically-related entities
- **`RealityView` post-processing** — `customPostProcessing` with Metal Performance Shaders, `CIFilter`, or custom shaders (iOS, iPadOS, macOS, tvOS)

## APIs & Frameworks

### RealityKit
- **`SpatialTrackingSession`** (expanded) — `Configuration(tracking:sceneUnderstanding:)` **[NEW flags]**
- `SpatialTrackingSession.Configuration.SceneUnderstandingFlags` — `.collision`, `.physics` **[NEW]**
- **`AnchorStateEvents.DidAnchor`** **[NEW]** — event with `entity` containing `ARKitAnchorComponent`
- **`AnchorStateEvents.WillUnanchor`** **[NEW]**
- **`AnchorStateEvents.DidFailToAnchor`** **[NEW]**
- **`ARKitAnchorComponent`** **[NEW]** — component holding raw ARKit anchor data
- `AnchorEntity(.plane(_:classification:minimumBounds:))` (existing, expanded)
- **`ManipulationComponent`** **[NEW]** — `configureEntity(_:collisionShapes:)`, `releaseBehavior`
- `ManipulationComponent.ReleaseBehavior` — `.animateToStart`, `.stay` **[NEW]**
- **`ManipulationEvents.WillBegin`** / `.WillEnd` / `.DidUpdateTransform` / `.DidHandOff` / `.WillRelease` **[NEW]**
- **`EnvironmentBlendingComponent`** **[NEW]** — `preferredBlendingMode: .occluded(by: .surroundings)`
- **`MeshInstancesComponent`** **[NEW]** — `[partIndex: Int]` subscript, `LowLevelInstanceData`
- **`LowLevelInstanceData(instanceCount:)`** **[NEW]** — `withMutableTransforms(_:)`
- **`ImagePresentationComponent`** **[NEW]** — `init(contentsOf:)` async, `desiredViewingMode`, `availableViewingModes`
- `ImagePresentationComponent.ViewingMode` — `.monoscopic`, `.spatialStereo`, `.spatialStereoImmersive`, `.spatial3D`, `.spatial3DImmersive` **[NEW]**
- **`ImagePresentationComponent.Spatial3DImage`** **[NEW]** — `init(contentsOf:)`, `generate()` async
- `VideoPlayerComponent` — expanded immersive video format support **[NEW]**
- **`ViewAttachmentComponent`** **[NEW]**
- **`PresentationComponent`** **[NEW]**
- **`GestureComponent`** **[NEW]**
- `Entity.attach(to: EntityPin)` **[NEW]**
- **`Entity(from: Data)`** **[NEW]** — async initializer from in-memory data
- AVIF texture support **[NEW]**
- `HoverEffectComponent` — `groupID` property **[NEW]**
- `RealityView.customPostProcessing(_:)` **[NEW]** — iOS, iPadOS, macOS, tvOS

## Code Highlights

```swift
// ARKit anchor via RealityKit
let planeAnchor = AnchorEntity(.plane(.horizontal, classification: .table, minimumBounds: [0.15, 0.15]))
content.subscribe(to: AnchorStateEvents.DidAnchor.self) { event in
    guard let anchor = event.entity.components[ARKitAnchorComponent.self]?.anchor as? PlaneAnchor else { return }
    gameRoot.transform = Transform(matrix: anchor.originFromAnchorTransform * anchor.geometry.extent.anchorFromExtentTransform)
}

// ManipulationComponent — one-line setup
ManipulationComponent.configureEntity(entity, collisionShapes: [shape])

// MeshInstancesComponent
var meshInstances = MeshInstancesComponent()
let instances = try LowLevelInstanceData(instanceCount: 20)
meshInstances[partIndex: 0] = instances
instances.withMutableTransforms { transforms in
    for i in 0..<20 { transforms[i] = randomTransform().matrix }
}
entity.components.set(meshInstances)

// Spatial scene generation
let spatial3DImage = try await ImagePresentationComponent.Spatial3DImage(contentsOf: url)
var component = ImagePresentationComponent(spatial3DImage: spatial3DImage)
component.desiredViewingMode = .spatial3D
entity.components.set(component)
try await spatial3DImage.generate()
```

## Takeaways
- Use `AnchorStateEvents` + `ARKitAnchorComponent` to access raw ARKit anchor geometry without needing a separate ARKit session — this is simpler for most AR anchoring use cases.
- `ManipulationComponent.configureEntity` is the fastest path to pick-up/rotate interactions in visionOS; toggle `PhysicsBodyComponent.mode` between `.kinematic` and `.dynamic` in `WillBegin`/`WillEnd` events to combine manipulation with physics.
- Use `MeshInstancesComponent` whenever you need 10+ copies of the same mesh — it reduces GPU data bandwidth significantly vs. cloning entities.
- `ImagePresentationComponent` is the new standard path for presenting spatial content (photos, spatial scenes) in a RealityKit scene; spatial scene generation is async and can be triggered lazily on user action.

---
_Source: WWDC25 Session 287 page (abstract, chapter summaries, code samples, and resource links)._
