# Introducing Enterprise APIs for visionOS
**WWDC24 · Session 10139** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10139/)

_Platforms:_ visionOS 2

## Overview
Apple is introducing six new enterprise-only APIs for visionOS that unlock significantly deeper device capabilities for business applications. These APIs require managed entitlements and a license file tied to the developer account, and are restricted to proprietary in-house apps or apps privately distributed through Apple Business Manager — they are not available for App Store distribution.

The APIs fall into two categories: enhanced sensor access (main camera feed, passthrough in screen captures, spatial barcode/QR scanning) and increased platform control (Apple Neural Engine access, object tracking parameter tuning, and increased CPU/GPU performance headroom). A demo shows all three sensor APIs combined into a support-center app scenario.

## Key Topics

**Enhanced Sensor Access**

*Main Camera Access* — `main-camera-access.allow` entitlement
- Provides access to the device's main camera video feed via `CameraFrameProvider` with `CameraVideoFormat` for the `.main` format
- Frames arrive as `CameraFrameUpdates` async sequence; each frame has a `.left` sample (the main camera is on the left side)
- Use cases: computer vision for defect detection, remote collaboration, quality assurance

*Passthrough in Screen Capture* — `include-passthrough` entitlement
- When capturing the screen, the normally-black background is replaced with the passthrough camera view
- Requires a Broadcast Upload Extension and ReplayKit's system "Start Broadcast" button (preserves user privacy)
- Use cases: "see what I see" remote support, field technician guidance

*Spatial Barcode and QR Code Scanning* — `barcode-detection.allow` entitlement
- `BarcodeDetectionProvider` automatically detects and tracks barcodes in 3D space
- Provides event-based updates: `.added`, `.updated`, `.removed` anchor events
- Supported symbologies: Code-39, QR, UPC-E, and others (see documentation)
- Requires `.worldSensing` ARKit authorization
- Use cases: warehouse logistics, inventory management, parts identification

**Platform Control**

*Apple Neural Engine Access* — `neural-engine-access` entitlement
- Unlocks the Apple Neural Engine as a compute device for Core ML models on visionOS (previously CPU/GPU only)
- Use `MLModel.availableComputeDevices` to verify access; Core ML dynamically routes model computation to the most efficient device
- Falls back to GPU/CPU if Neural Engine is unavailable or less efficient for the workload

*Object Tracking Parameter Adjustment* — `object-tracking-parameter-adjustment.allow` entitlement
- Enhances the new known-object tracking feature in visionOS 2.0 with configurable parameters
- `ObjectTrackingProvider.TrackingConfiguration`: `maximumTrackableInstances` (default 10, up to 15+), `maximumInstancesPerReferenceObject`, `detectionRate`, `stationaryObjectTrackingRate`, `movingObjectTrackingRate`
- Increasing one parameter may require reducing others to balance compute load
- Reference objects are `.referenceObject` files created with Apple's tooling

*App Compute Settings* — `app-compute-category` entitlement
- Increases CPU and GPU performance headroom in exchange for higher fan speed and slightly reduced battery life
- App-wide setting, no API to toggle at runtime; system activates automatically based on workload
- Suitable for apps rendering highly complex 3D content or running intensive compute pipelines

## APIs & Frameworks

**ARKit (visionOS)**
- `CameraFrameProvider` **[NEW]** — provides access to camera video frames
- `CameraVideoFormat.supportedVideoFormats(for:cameraPositions:)` **[NEW]** — list supported formats for `.main` camera
- `CameraFrameProvider.cameraFrameUpdates(for:)` **[NEW]** — async sequence of camera frames
- `ARKitSession` — session used to run providers
- `ARKitSession.queryAuthorization(for:)` — request `.cameraAccess`, `.worldSensing` permissions
- `BarcodeDetectionProvider(symbologies:)` **[NEW]** — detects and tracks barcodes
- `BarcodeDetectionProvider.anchorUpdates` **[NEW]** — async sequence of barcode anchor events (`.added`, `.updated`, `.removed`)
- `ObjectTrackingProvider` — track known reference objects in the scene
- `ObjectTrackingProvider.TrackingConfiguration` **[NEW enterprise]** — configure tracking parameters
- `TrackingConfiguration.maximumTrackableInstances` **[NEW enterprise]**
- `TrackingConfiguration.detectionRate`, `stationaryObjectTrackingRate`, `movingObjectTrackingRate` **[NEW enterprise]**

**Core ML**
- `MLModel.availableComputeDevices` — list available compute devices (`.cpu`, `.gpu`, `.neuralEngine`)
- Neural Engine unlocked as compute device with `neural-engine-access` entitlement **[NEW enterprise]**

**ReplayKit**
- Broadcast Upload Extension required for passthrough screen capture
- System "Start Broadcast" button triggers capture flow

**Entitlements (Enterprise Only)**
- `main-camera-access.allow`
- `include-passthrough`
- `barcode-detection.allow`
- `neural-engine-access`
- `object-tracking-parameter-adjustment.allow`
- `app-compute-category`

## Code Highlights

Main camera access:
```swift
let formats = CameraVideoFormat.supportedVideoFormats(for: .main, cameraPositions: [.left])
let cameraFrameProvider = CameraFrameProvider()
try await arKitSession.run([cameraFrameProvider])
for await cameraFrame in cameraFrameProvider.cameraFrameUpdates(for: formats[0])! {
    pixelBuffer = cameraFrame.sample(for: .left)?.pixelBuffer
}
```

Barcode detection:
```swift
let barcodeDetection = BarcodeDetectionProvider(symbologies: [.code39, .qr, .upce])
try await arkitSession.run([barcodeDetection])
for await anchorUpdate in barcodeDetection.anchorUpdates {
    switch anchorUpdate.event {
    case .added: addEntity(for: anchorUpdate.anchor)
    case .updated: updateEntity(for: anchorUpdate.anchor)
    case .removed: removeEntity(for: anchorUpdate.anchor)
    }
}
```

## Takeaways
- Enterprise APIs require managed entitlements and are restricted to in-house or Business Manager-distributed apps — not the App Store.
- Apply for only the specific entitlements your app needs; Apple evaluates each application individually.
- Use `BarcodeDetectionProvider` and `CameraFrameProvider` as building blocks; combine them to create rich enterprise spatial experiences.
- Always consider employee privacy, especially when using main camera access and passthrough capture in workplace environments.

---
_Source: WWDC24 Session 10139 page (abstract, chapter summaries, code samples, and resource links)._
