# Evolve Your ARKit App for Spatial Experiences
**WWDC23 · Session 10091** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10091/)

_Platforms:_ visionOS 1, iOS 17

## Overview
This session is a conceptual bridge for iOS ARKit developers moving their apps to visionOS. It explains how ARKit and RealityKit have fundamentally changed on the new platform: many responsibilities that used to belong to apps (camera passthrough, matting, world map persistence, session management) are now handled by system services. The app developer is freed to focus on content and experience.

Key architectural shifts include the replacement of `ARSession` configurations with modular data providers, the end of `ARFrame` delivery (the system handles passthrough), a new provider-per-capability model for scene understanding, and system-managed world anchor persistence. `RealityView` replaces `ARView` as the SwiftUI/RealityKit bridge, and `ImmersiveSpace` replaces the need for full-screen AR camera views.

The session also covers two approaches to content placement: using `AnchorEntity` (no permission required, no transform exposed) and using ARKit data providers with `WorldAnchor` for full custom placement logic.

## Key Topics

- **Platform presentation model** — Shared Space (side-by-side with other apps, windows and volumes), Full Space (exclusive, enables ARKit access and `AnchorEntity` on surroundings), `ImmersiveSpace` SwiftUI scene type.
- **Content tools** — USD as the foundation; Reality Composer Pro for composing, editing, and previewing; ShaderGraph replaces CustomMaterials; Reality Composer Pro projects importable into Xcode.
- **RealityView vs. ARView** — `RealityView` is the SwiftUI container replacing `ARView`; uses `Content` instead of `Scene`; `AnchorEntity` works without permissions (no transforms exposed to app); gesture support via SwiftUI gestures; entities added to `Content` share a coordinate space.
- **ARKit data provider model** — à la carte provider selection replacing preset configurations; `SceneReconstructionProvider` for mesh anchors, `PlaneDetectionProvider` for plane anchors, `HandTrackingProvider` for hands, `WorldTrackingProvider` for world anchors; providers deliver typed anchors directly without disambiguation; no `ARFrame` delivery.
- **Raycasting** — collision components from mesh anchors enable hit testing; `SpatialTapGesture` for system-gesture raycasting; hand joint positions for custom ray construction; `CollisionCastHit` for results; `WorldAnchor` to persist placement.
- **World anchor persistence** — system continuously persists world map in background; apps use `WorldTrackingProvider` to add/remove `WorldAnchor` objects; no save/load/relocalization code required.

## APIs & Frameworks

**ARKit (visionOS)**
- `ARKitSession` **[NEW on visionOS]** — runs data providers; replaces `ARSession`
- `SceneReconstructionProvider` **[NEW]** — delivers `MeshAnchor` updates
- `PlaneDetectionProvider` **[NEW]** — delivers `PlaneAnchor` updates; configured with plane types
- `HandTrackingProvider` **[NEW]** — delivers hand skeleton anchor updates
- `WorldTrackingProvider` **[NEW]** — delivers `WorldAnchor` updates; manages world anchor persistence
- `WorldAnchor` **[NEW]** — ARKit-tracked world-space location; system persists automatically
- `MeshAnchor` **[NEW]** — scene reconstruction mesh chunk with geometry and semantic data
- `PlaneAnchor` **[NEW]** — detected plane with classification and bounds
- `HandAnchor` / hand joint data **[NEW]** — finger joint positions for raycasting and interaction

**RealityKit (visionOS)**
- `RealityView` **[NEW]** — SwiftUI view bridging RealityKit; replaces `ARView`
- `RealityView.Content` — entity container; replaces `ARView.Scene`
- `AnchorEntity` — attach content to surroundings (wall, table, palm, wrist) without permissions; no transform exposed to app
- `AnchorEntity(.plane(_:classification:minimumBounds:))` — surface-type anchor specification
- `InputTargetComponent` — marks entity as gesture-targetable
- `CollisionComponent` — collision shape for hit testing and physics
- `ShapeResource.generateStaticMesh(from:)` — generate collision shape from mesh anchor geometry
- `SpatialTapGesture` — system gesture for look + tap raycasting
- `CollisionCastHit` — result of a raycast including entity, position, normal

**SwiftUI (visionOS)**
- `ImmersiveSpace` **[NEW]** — SwiftUI scene type for full-room spatial content; required for ARKit data access
- `.immersionStyle(selection:in:)` modifier — `.mixed`, `.full`, `.progressive`
- `WindowGroup` / `VolumetricWindowGroup` — windowed and 3D volume scene types

**Reality Composer Pro**
- ShaderGraph — replaces CustomMaterials from iOS ARKit apps
- Component editor — visual editing of RealityKit components (CollisionComponent, etc.)
- USD asset import and preview

## Code Highlights

New ARKit session setup (à la carte providers):
```swift
let meshProvider = SceneReconstructionProvider(modes: [.classification])
let planeProvider = PlaneDetectionProvider(alignments: [.horizontal, .vertical])
let session = ARKitSession()
try await session.run([meshProvider, planeProvider])

for await anchor in meshProvider.anchorUpdates {
    // anchor is typed MeshAnchor — no disambiguation needed
}
```

Building a collision entity from a mesh anchor for raycasting:
```swift
let entity = Entity()
entity.transform = Transform(matrix: meshAnchor.originFromAnchorTransform)
let shape = try await ShapeResource.generateStaticMesh(from: meshAnchor)
entity.components[CollisionComponent.self] = CollisionComponent(shapes: [shape])
entity.components[InputTargetComponent.self] = InputTargetComponent()
content.add(entity)
```

System-gesture raycasting with SpatialTapGesture:
```swift
RealityView { content in /* ... */ }
    .gesture(SpatialTapGesture().targetedToAnyEntity().onEnded { value in
        let worldPosition = value.location3D
        // Use worldPosition to place a WorldAnchor
    })
```

## Takeaways

- The system now handles camera passthrough, hand matting, and world map persistence — apps no longer manage these; this simplifies the codebase considerably.
- Replace `ARSession` + configuration with `ARKitSession` + individual data providers; each provider delivers strongly typed anchors with no need for casting.
- Use `AnchorEntity` in a Full Space for permission-free placement; use ARKit's `WorldTrackingProvider` + `WorldAnchor` when you need custom placement logic or persistence.
- `ARFrame` is gone — camera frames are system-managed; build spatial content with `RealityView` and `ImmersiveSpace` instead.

---
_Source: WWDC23 Session 10091 page (abstract, chapter summaries, code samples, and resource links)._
