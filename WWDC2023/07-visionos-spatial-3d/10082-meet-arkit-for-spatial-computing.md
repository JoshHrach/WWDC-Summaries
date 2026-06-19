# Meet ARKit for Spatial Computing
**WWDC23 · Session 10082** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10082/)

_Platforms:_ visionOS 1

## Overview
ARKit has been rebuilt from the ground up for visionOS as a full system service rather than an app-level framework. It now powers every interaction on the platform — from window manipulation to immersive games — and exposes a fully redesigned API available in both modern Swift and classic C. The new design offers features à la carte: apps compose exactly the data providers they need rather than enabling a monolithic session configuration.

This session covers the four pillars of the new ARKit for visionOS: world tracking (placing virtual content in the real world), scene understanding (planes, scene geometry, image tracking), hand tracking (a new feature for direct spatial input), and a full "TimeForCube" demo that combines scene reconstruction with hand collision for physics-based interaction.

## Key Topics

### Architecture and Privacy
- Sensor data (camera frames, etc.) is never sent to client space; it goes to ARKit's secure daemon for processing.
- ARKit curates and forwards only the minimal derived data (anchor transforms, skeleton joints) to apps.
- Apps must be in a **Full Space** (ImmersiveSpace) to receive ARKit data; Shared Space apps receive nothing.
- Some data types (e.g., hand tracking) require explicit user authorization.

### Core Building Blocks
- **Anchor** — a position/orientation in the real world. Includes `id` and `transform`; some are `TrackableAnchor` with an `isTracked` bool.
- **DataProvider** — represents one ARKit feature; exposes an async sequence of anchor updates or a poll API.
- **ARKitSession** — runs a set of data providers together; manages authorization and lifecycle.

### Authorization
- `ARKitSession.requestAuthorization(for:)` — batch-request authorization for required data types before running.
- `ARKitSession.AuthorizationType` — e.g., `.handTracking`.
- Authorization status: `.allowed`, `.denied`, `.notDetermined`.
- Running a session with a denied provider causes the session to fail.

### World Tracking
- `WorldTrackingProvider` — tracks the device in 6DoF; lets apps add/query world anchors.
- `WorldAnchor` — `TrackableAnchor` placed at a specific transform in the real world; persisted automatically across app launches and reboots.
- WorldAnchor persistence is map-based and location-aware (home map vs. office map).
- Only anchor IDs and transforms are persisted; apps maintain their own mapping of anchor IDs to virtual content.
- `WorldTrackingProvider.queryDevicePose(atTimestamp:)` — used by Metal/CompositorServices renderers to get the device transform per frame.
- `cp_drawable_set_ar_pose(_:_:)` — tells CompositorServices which pose was used to render the frame.

### Scene Understanding
**Plane Detection (`PlaneDetectionProvider` / `PlaneAnchor`):**
- Detects horizontal and vertical surfaces; provides alignment, geometry, and semantic classification.
- Classifications: `.floor`, `.table`, `.wall`, `.ceiling`, `.seat`, `.door`, `.window`, `.unknown`, `.undetermined`, `.notAvailable`.
- Useful for content placement on flat surfaces and basic physics colliders.

**Scene Geometry (`SceneReconstructionProvider` / `MeshAnchor`):**
- Reconstructs surroundings as a subdivided polygonal mesh.
- `MeshAnchor` geometry: vertices, normals, faces, per-face semantic classifications.
- Classifications: `.wall`, `.floor`, `.ceiling`, `.table`, `.seat`, `.window`, `.door`, `.none`.
- Enables high-fidelity physics and interaction with non-flat real-world objects.

**Image Tracking (`ImageTrackingProvider` / `ImageAnchor`):**
- Detects 2D reference images in the real world.
- `ReferenceImage` — loaded from AR resource group in asset catalog, or from `CVPixelBuffer`/`CGImage`.
- `ImageAnchor` — `TrackableAnchor` with `estimatedScaleFactor` comparing detected to specified physical size.

### Hand Tracking (New)
- `HandTrackingProvider` — new visionOS feature; provides skeletal hand data.
- `HandAnchor` — `TrackableAnchor` with `chirality` (`.left`/`.right`) and a `Skeleton`.
- `HandAnchor.transform` — the wrist's transform relative to the app origin.
- `Skeleton.joint(named:)` — returns a `Skeleton.Joint` by `SkeletonDefinition.JointName`.
- `Skeleton.Joint` — has `parentJoint`, `name`, `localTransform` (relative to parent), `rootTransform` (relative to root/wrist), `isTracked`.
- Wrist is the skeleton root; finger joints parented sequentially outward.
- Two access patterns: async updates (`handTracking.anchorUpdates`) or poll in render loop (`ar_hand_tracking_provider_get_latest_anchors`).
- Hand occlusion (system feature): virtual content is occluded by the user's hands by default.
- `upperLimbVisibility` setter on scene — controls whether hands occlude virtual content.

### Practical Demo: TimeForCube
- `ImmersiveSpace` (required for ARKit access) containing a `RealityView`.
- `SceneReconstructionProvider` used to build mesh colliders (`ShapeResource.generateStaticMesh(from:)`); each mesh entity gets `CollisionComponent`, `PhysicsBodyComponent`, and `InputTargetComponent`.
- `HandTrackingProvider` used to drive invisible 5mm sphere entities (fingertip colliders) with kinematic physics bodies.
- `SpatialTapGesture().targetedToAnyEntity()` detects taps on mesh/cube entities; converts tap location from global to scene coordinates.
- Cubes spawned 20cm above tap location with indirect-only `InputTargetComponent`; physics handles the rest.

## APIs & Frameworks
- `ARKit` framework (visionOS) **[NEW architecture]** — system service for tracking and scene understanding
- `ARKitSession` **[NEW]** — manages data providers; handles authorization
- `ARKitSession.requestAuthorization(for:)` **[NEW]** — batch authorization request
- `ARKitSession.AuthorizationType` **[NEW]** — authorization kinds (`.handTracking`, etc.)
- `WorldTrackingProvider` **[NEW]** — 6DoF device tracking and world anchoring
- `WorldAnchor` **[NEW]** — persistent real-world anchor; `TrackableAnchor`
- `PlaneDetectionProvider` **[NEW]** — horizontal/vertical surface detection
- `PlaneAnchor` **[NEW]** — flat surface anchor with `alignment`, `geometry`, `classification`
- `PlaneAnchor.Classification` **[NEW]** — semantic surface type enum
- `SceneReconstructionProvider` **[NEW]** — polygonal mesh reconstruction of surroundings
- `MeshAnchor` **[NEW]** — mesh anchor with `geometry` (vertices, faces, per-face classifications)
- `MeshAnchor.MeshClassification` **[NEW]** — per-face semantic classification enum
- `ShapeResource.generateStaticMesh(from:)` **[NEW]** — generates a `ShapeResource` from a `MeshAnchor` for physics/collision
- `ImageTrackingProvider` **[NEW]** — 2D reference image detection
- `ImageAnchor` **[NEW]** — `TrackableAnchor` for detected reference images; `estimatedScaleFactor`
- `ReferenceImage` **[NEW]** — loadable from asset catalog or `CGImage`/`CVPixelBuffer`
- `HandTrackingProvider` **[NEW]** — skeletal hand tracking
- `HandAnchor` **[NEW]** — `TrackableAnchor` for each hand; `chirality`, `skeleton`, `transform`
- `HandAnchor.Chirality` **[NEW]** — `.left`, `.right`
- `Skeleton` **[NEW]** — hand skeleton; `joint(named:)`
- `Skeleton.Joint` **[NEW]** — `localTransform`, `rootTransform`, `isTracked`, `parentJoint`
- `SkeletonDefinition.JointName` **[NEW]** — named joint identifiers (e.g., `.handIndexFingerTip`)
- `AnchorUpdate` **[NEW]** — wrapper around an anchor update event (`.added`, `.updated`, `.removed`)
- `InputTargetComponent` (RealityKit) — marks entity as gesture target; `allowedInputTypes`
- `CollisionComponent` (RealityKit) — collision shape for physics/gesture
- `PhysicsBodyComponent` (RealityKit) — physics simulation component
- `SpatialTapGesture` (SwiftUI/RealityKit) — 3D tap gesture; `.targetedToAnyEntity()`
- `CompositorServices` / `cp_drawable_set_ar_pose` — C API for Metal rendering with ARKit poses
- `ar_session_t`, `ar_world_tracking_provider_t`, `ar_hand_tracking_provider_t` — C API equivalents

## Code Highlights

Request hand tracking authorization:
```swift
let session = ARKitSession()
Task {
    let authorizationResult = await session.requestAuthorization(for: [.handTracking])
    for (authorizationType, authorizationStatus) in authorizationResult {
        switch authorizationStatus {
        case .allowed: break  // proceed
        case .denied: /* handle */ break
        default: break
        }
    }
}
```

Process hand anchor updates (async):
```swift
func processHandUpdates() async {
    for await update in handTracking.anchorUpdates {
        let handAnchor = update.anchor
        guard handAnchor.isTracked else { continue }
        let fingertip = handAnchor.skeleton.joint(named: .handIndexFingerTip)
        guard fingertip.isTracked else { continue }
        let originFromWrist = handAnchor.transform
        let wristFromIndex = fingertip.rootTransform
        let originFromIndex = originFromWrist * wristFromIndex
        fingerEntities[handAnchor.chirality]?.setTransformMatrix(originFromIndex, relativeTo: nil)
    }
}
```

Process scene reconstruction updates:
```swift
func processReconstructionUpdates() async {
    for await update in sceneReconstruction.anchorUpdates {
        let meshAnchor = update.anchor
        guard let shape = try? await ShapeResource.generateStaticMesh(from: meshAnchor) else { continue }
        switch update.event {
        case .added:
            let entity = ModelEntity()
            entity.transform = Transform(matrix: meshAnchor.transform)
            entity.collision = CollisionComponent(shapes: [shape], isStatic: true)
            entity.physicsBody = PhysicsBodyComponent()
            entity.components.set(InputTargetComponent())
            meshEntities[meshAnchor.id] = entity
            contentEntity.addChild(entity)
        case .updated:
            meshEntities[meshAnchor.id]?.collision?.shapes = [shape]
        case .removed:
            meshEntities[meshAnchor.id]?.removeFromParent()
            meshEntities.removeValue(forKey: meshAnchor.id)
        @unknown default: fatalError()
        }
    }
}
```

## Takeaways
- ARKit on visionOS is a system-wide service with a privacy-by-default architecture: raw sensor data stays in the daemon; only curated anchor/joint data reaches your app.
- Apps must be in a Full Space (`ImmersiveSpace`) to use ARKit; compose exactly the data providers needed using `ARKitSession`.
- `WorldAnchor` persistence is automatic and map-based — only store anchor IDs alongside your content; anchors relocalize when the user returns to the same physical location.
- Hand tracking (`HandTrackingProvider`) enables direct spatial interaction by placing kinematic physics bodies at fingertip joints; combine with `SceneReconstructionProvider` mesh colliders for fully physical mixed-reality experiences.

---
_Source: WWDC23 Session 10082 page (abstract, transcript, chapter summaries, and code samples)._
