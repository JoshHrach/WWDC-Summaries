# Introducing ARKit 3
**WWDC19 · Session 604** · [Watch](https://developer.apple.com/videos/play/wwdc2019/604/)

_Platforms:_ iOS 13, iPadOS 13 (A12 Bionic or later required for people occlusion, motion capture, and simultaneous front/back camera)

## Overview
ARKit 3 represents the largest leap forward since ARKit's debut in 2017, introducing people occlusion, full-body motion capture, collaborative multi-user sessions, simultaneous front and back camera tracking, multiple face tracking, and a built-in AR coaching UI. These features are powered by Apple's Neural Engine and machine learning, enabling entirely new categories of AR experiences that previously required specialized equipment or custom infrastructure.

The session is companion to the new RealityKit framework, which is designed from the ground up for AR rendering. Many of the new ARKit 3 APIs integrate directly with RealityKit's `ARView` and entity hierarchy, reducing the amount of setup code significantly compared to prior releases.

Scene understanding also received major upgrades: plane estimation now uses ML to detect planes even without feature points, image detection scales to 100 simultaneous images, object detection is faster and more environment-tolerant, and a new raycasting API enables tracked, continuously updated surface placement. A new record-and-replay workflow in Reality Composer lets developers capture sensor data in the field and replay it in Xcode at their desk.

## Key Topics

**People Occlusion**
ARKit 3 uses ML running on the Apple Neural Engine to segment people in the camera frame and estimate their depth relative to virtual objects, enabling virtual content to be rendered correctly behind foreground people. Activated via a `frameSemantics` property on `ARConfiguration` with two modes: `personSegmentation` (layer always in front) and `personSegmentationWithDepth` (depth-correct ordering).

**Motion Capture (2D and 3D)**
Detects and tracks a human body skeleton both in normalized 2D image space (`ARBody2D`) and in 3D world coordinates (`ARBodyAnchor`). The 3D skeleton includes scale estimation and is anchored in world coordinates. Joints are provided in a flat array with a `ARSkeletonDefinition` describing hierarchy and named joints. RealityKit's `BodyTrackedEntity` can drive a rigged mesh from the skeleton automatically.

**Collaborative Sessions**
ARKit 3 continuously shares mapping data and user-created `ARAnchor`s across multiple devices over a network (via Multipeer Connectivity or any transport). Each anchor is tagged with a session identifier. `ARParticipantAnchor` provides real-time positions of other participants. Enabled by setting `isCollaborationEnabled = true` on `ARWorldTrackingConfiguration` and implementing an `ARSessionDelegate` method to forward `ARCollaborationData` packets.

**Simultaneous Front and Back Camera**
`ARWorldTrackingConfiguration` now supports `userFaceTrackingEnabled`, and `ARFaceTrackingConfiguration` supports `worldTrackingEnabled`, allowing both cameras to be used concurrently on A12+ devices. Face transforms in world tracking are placed behind the device and must be translated to appear in the scene.

**Multiple Face Tracking**
`ARFaceTrackingConfiguration` can now track up to three faces simultaneously, with persistent face anchor IDs within a session. Two new properties: `maximumNumberOfTrackedFaces` (query device limit) and `maximumNumberOfTrackedFaces` (set desired count).

**AR Coaching Overlay**
`ARCoachingOverlayView` is a new system UI overlay that guides users through onboarding, relocalizing, and plane-finding. It automatically activates and deactivates in response to tracking state changes and provides consistent design across apps (matching AR Quick Look and Measure). Delegates surface activation/deactivation events and relocalization abort requests.

**Scene Understanding Improvements**
- Plane estimation now uses ML to detect planes and classify them even without feature points; two new plane classifications added: `.door` and `.window` (joining wall, floor, ceiling, table, seat from ARKit 2).
- New tracked raycasting API (`ARSession.trackedRaycast(query:updateHandler:)`) continuously updates object placement as scene understanding improves.
- Image detection now supports up to 100 simultaneous reference images and automatic scale estimation for printed images.
- `ARReferenceImage.validate(completionHandler:)` lets developers check image quality at runtime.
- Object detection enhanced with ML for faster and more environment-tolerant recognition.

**New Configuration: ARPositionalTrackingConfiguration**
Lightweight configuration for tracking-only use cases (no camera backdrop rendering), with reduced capture frame rate and resolution for lower power consumption while maintaining 60 Hz rendering.

**Visual Coherence Enhancements**
- Depth of field matching between virtual content and physical scene.
- Camera motion blur applied to virtual objects via `ARView` render options.
- HDR environment textures for more vibrant reflections in bright environments.
- Camera grain API to match sensor noise on virtual objects in low light.

## APIs & Frameworks

**ARKit** (ARKit 3) **[NEW features]**
- `ARConfiguration.frameSemantics: ARConfiguration.FrameSemantics` **[NEW]**
  - `.personSegmentation` **[NEW]**
  - `.personSegmentationWithDepth` **[NEW]**
  - `.bodyDetection` **[NEW]**
- `ARConfiguration.supportsFrameSemantics(_:)` class method **[NEW]**
- `ARFrame.segmentationBuffer: CVPixelBuffer?` **[NEW]**
- `ARFrame.estimatedDepthData: CVPixelBuffer?` **[NEW]**
- `ARBody2D` — 2D body detection result **[NEW]**
- `ARFrame.detectedBody: ARBody2D?` **[NEW]**
- `ARSkeleton2D` — 2D normalized joint positions **[NEW]**
- `ARSkeletonDefinition` — hierarchy and named joint definitions **[NEW]**
- `ARBodyTrackingConfiguration` — new 3D body tracking configuration **[NEW]**
- `ARBodyAnchor` — anchor for a tracked 3D body **[NEW]**
- `ARBodyAnchor.skeleton: ARSkeleton3D` **[NEW]**
- `ARBodyAnchor.estimatedScaleFactor: Float` **[NEW]**
- `ARSkeleton3D` — 3D joint transforms in world space **[NEW]**
- `ARCollaborationData` — sharable collaboration packets **[NEW]**
- `ARWorldTrackingConfiguration.isCollaborationEnabled: Bool` **[NEW]**
- `ARSessionDelegate.session(_:didOutputCollaborationData:)` **[NEW]**
- `ARSession.update(with:)` — ingest received `ARCollaborationData` **[NEW]**
- `ARParticipantAnchor` — real-time participant position in collaborative session **[NEW]**
- `ARWorldTrackingConfiguration.userFaceTrackingEnabled: Bool` **[NEW]**
- `ARFaceTrackingConfiguration.isWorldTrackingEnabled: Bool` **[NEW]**
- `ARFaceTrackingConfiguration.maximumNumberOfTrackedFaces: Int` **[NEW]**
- `ARCoachingOverlayView` — built-in onboarding UI **[NEW]**
- `ARCoachingOverlayViewDelegate` **[NEW]**
  - `coachingOverlayViewDidActivate(_:)` **[NEW]**
  - `coachingOverlayViewDidDeactivate(_:)` **[NEW]**
  - `coachingOverlayViewDidRequestSessionReset(_:)` **[NEW]**
- `ARCoachingOverlayView.goal: ARCoachingOverlayView.Goal` **[NEW]**
- `ARPositionalTrackingConfiguration` — lightweight tracking-only config **[NEW]**
- `ARRayCastQuery` **[NEW]**
- `ARSession.trackedRaycast(query:updateHandler:) -> ARTrackedRaycast` **[NEW]**
- `ARTrackedRaycast.stopTracking()` **[NEW]**
- `ARRayCastQuery.Target` — `.existingPlaneGeometry`, `.estimatedPlane`, `.infinite` **[NEW]**
- `ARRayCastQuery.TargetAlignment` — `.horizontal`, `.vertical`, `.any` **[NEW]**
- `ARPlaneAnchor.Classification` additions: `.door`, `.window` **[NEW]**
- `ARReferenceImage.validate(completionHandler:)` **[NEW]**
- `ARSession.run(configuration:)` (existing, now supports new configs)
- `ARWorldTrackingConfiguration` (existing)
- `ARFaceTrackingConfiguration` (existing)
- `ARWorldMap` (existing, ARKit 2)

**RealityKit** (new in 2019)
- `ARView` — render view with AR integration **[NEW]**
- `ARView.renderOptions: ARView.RenderOptions` **[NEW]**
- `AnchorEntity(tracking: .body)` **[NEW]**
- `BodyTrackedEntity` — drives rigged mesh from `ARBodyAnchor` skeleton **[NEW]**
- `ModelEntity.loadAsync(named:)` **[NEW]**
- `ARView.scene.synchronizationService` — collaborative session binding **[NEW]**

**MultipeerConnectivity** (existing, used for collaborative sessions)
- `MCSession`
- `MCNearbyServiceAdvertiser` / `MCNearbyServiceBrowser`

## Code Highlights

Enabling people occlusion:
```swift
guard ARWorldTrackingConfiguration.supportsFrameSemantics(.personSegmentationWithDepth) else {
    // Not available on this device
    return
}
configuration.frameSemantics.insert(.personSegmentationWithDepth)
arView.session.run(configuration)
```

Setting up 3D body tracking with RealityKit character:
```swift
let bodyAnchor = AnchorEntity(.body)
arView.scene.addAnchor(bodyAnchor)

ModelEntity.loadAsync(named: "robot").sink { character in
    bodyAnchor.addChild(character)
}.store(in: &cancellables)
```

Enabling collaborative sessions:
```swift
configuration.isCollaborationEnabled = true
arView.scene.synchronizationService = try! MultipeerConnectivityService(session: mcSession)
arView.session.run(configuration)
```

Tracked raycasting:
```swift
let query = ARRayCastQuery(origin: screenCenter, allowing: .estimatedPlane, alignment: .any)
let raycast = session.trackedRaycast(query: query) { results in
    guard let result = results.first else { return }
    virtualObject.transform = result.worldTransform
}
// When done:
raycast.stopTracking()
```

## Takeaways
- ARKit 3 enables people occlusion and 3D motion capture on A12+ devices with just a few lines of code — capabilities that previously required specialized hardware or custom ML pipelines.
- Collaborative sessions via `ARCollaborationData` make true multi-user AR significantly simpler; the session handles map merging and anchor sharing automatically.
- The new `ARCoachingOverlayView` provides a system-consistent onboarding experience for free, improving first-run AR quality without custom UI work.
- The tracked raycasting API replaces hit-testing with a continuously updated placement system, making surface-relative object placement far more accurate as scene understanding evolves.

---
_Source: WWDC19 Session 604 page (abstract, chapter summaries, code samples, and resource links)._
