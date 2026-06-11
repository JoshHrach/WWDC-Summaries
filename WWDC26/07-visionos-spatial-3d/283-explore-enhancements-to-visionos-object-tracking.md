# Explore enhancements to visionOS object tracking
**WWDC26 · Session 283** · [Watch](https://developer.apple.com/videos/play/wwdc2026/283/)

_Platforms:_ visionOS 27, iOS 27

## Overview
This session covers two sets of advances: enhancements to the existing visionOS object tracking API, and the expansion of spatial accessories to third-party developers. Object tracking now supports high-frame-rate tracking of moving/handheld objects (previously limited to stationary objects), arrives on iOS for the first time via `ARWorldTrackingConfiguration`, and gains a coordinate-space correction API for distinguishing rendered poses from metric poses. Spatial accessories — custom hardware devices tracked by Apple Vision Pro — open to third-party manufacturers in visionOS 27.

The session provides practical guidance for choosing between stationary (low-frame-rate) and moving (high-frame-rate) tracking modes, explains the extended training mode in Create ML for more robust reference objects, and walks through the iOS implementation pattern. The spatial accessories section covers design requirements (LED constellation, IMU, Bluetooth), the reference accessory bundle format, and the plug-and-play API using `GCSpatialAccessory` and `AccessoryTrackingProvider`.

## Key Topics

### Object Tracking Enhancements (visionOS 27)
- `ReferenceObject.Configuration.highFrameRateTrackingEnabled = true` **[NEW]**: enables tracking of moving/handheld objects on visionOS.
- Extended training mode in Create ML (`--training-mode extended --all-angles`) produces more robust reference objects for dynamic tracking.
- `myObjectAnchor.coordinateSpace(correction: .rendered)` — pose with rendering corrections applied (default for visual attachment).
- `myObjectAnchor.coordinateSpace(correction: .none)` — raw metric-space pose (for measurement/physics).

### Object Tracking on iOS (NEW)
- First-time support for object tracking on iOS via `ARWorldTrackingConfiguration`.
- `configuration.detectionObjects` — stationary detection (low frame rate); existing.
- `configuration.trackingObjects` **[NEW]**: high-frame-rate tracking of moving objects on iOS.
- Same `ARObjectAnchor` delegate pattern (`didAdd`, `didUpdate`, `didRemove`).
- `ARAnchorEntity(anchor:)` links ARKit anchors to RealityKit entities.

### Spatial Accessories
- Spatial accessories are physical devices with an LED constellation, IMU, and Bluetooth that Apple Vision Pro tracks in real time.
- Available to third-party manufacturers in visionOS 27 (previously Apple-only).
- Design requirements: LED count, spacing, and arrangement must meet MFi guidelines; reference accessory bundle `.referenceaccessory` describes the hardware.
- Debug tool available in Simulator for validating LED patterns.

### Building a Spatial Accessory App
- `GCSpatialAccessory.spatialAccessories` (GameController) **[NEW]**: discover connected accessories.
- `Accessory(device: GCSpatialAccessory)` **[NEW]**: wraps a device; resolves the `.referenceaccessory` bundle automatically.
- `AccessoryTrackingProvider(accessories:)` **[NEW]**: ARKit provider for spatial accessory tracking.
- `arkitSession.run([provider])` — starts tracking.
- `provider.updateAccessories([newAccessory])` **[NEW]**: hot-swap accessories without restarting the ARKit session.

## APIs & Frameworks

### ARKit — visionOS (updated)
- `ReferenceObject.Configuration` — `highFrameRateTrackingEnabled: Bool` **[NEW]**
- `ReferenceObject(from:configuration:)` async — existing; loads reference object with new config
- `ObjectAnchor.coordinateSpace(correction:)` **[NEW]**: `.rendered` / `.none`
- `AccessoryTrackingProvider(accessories:)` **[NEW]**
- `AccessoryTrackingProvider.updateAccessories(_:)` async **[NEW]**
- `Accessory(device: GCSpatialAccessory)` **[NEW]**: wraps a GameController spatial accessory

### ARKit — iOS (NEW)
- `ARWorldTrackingConfiguration.trackingObjects: Set<ARReferenceObject>` **[NEW]**
- `ARWorldTrackingConfiguration.detectionObjects` — existing
- `ARReferenceObject(archiveURL:)` — existing
- `ARObjectAnchor` — `isTracked` property — existing
- `ARSessionDelegate` — `session(_:didAdd:)`, `didUpdate:`, `didRemove:` — existing
- `AnchorEntity(anchor:)` (RealityKit) — existing

### GameController (updated)
- `GCSpatialAccessory` **[NEW]**: class representing a tracked spatial accessory device
- `GCSpatialAccessory.spatialAccessories` **[NEW]**: class property enumerating connected accessories

### Create ML / Command Line
- `xrun createml objecttracker --training-mode extended --all-angles` **[NEW]**: extended training for moving objects

### Documentation
- [Working with generic spatial accessories](https://developer.apple.com/documentation/visionOS/working-with-generic-spatial-accessories)
- [Preparing spatial accessories for tracking in your visionOS app](https://developer.apple.com/documentation/ARKit/preparing-spatial-accessories-for-tracking-in-your-visionos-app)
- [Exploring object tracking with ARKit](https://developer.apple.com/documentation/visionOS/exploring_object_tracking_with_arkit)

## Code Highlights

Enable high-frame-rate tracking of a moving object:
```swift
var configuration = ReferenceObject.Configuration()
configuration.highFrameRateTrackingEnabled = true
let refObject = try? await ReferenceObject(from: refObjURL!, configuration: configuration)
```

Track objects on iOS:
```swift
let configuration = ARWorldTrackingConfiguration()
configuration.detectionObjects = [stationaryObject]  // low frame rate
configuration.trackingObjects = [movingObject]       // high frame rate
arView.session.run(configuration)
```

Discover and use a spatial accessory:
```swift
if let device = GCSpatialAccessory.spatialAccessories.first {
    let accessory = try await Accessory(device: device)
    let provider = AccessoryTrackingProvider(accessories: [accessory])
    try await arkitSession.run([provider])
}
```

## Takeaways
- High-frame-rate object tracking for moving objects is a significant upgrade — physical controllers, props, and hand-held tools can now be tracked reliably in dynamic applications.
- Object tracking on iOS opens a new platform dimension: the same reference objects built for visionOS work on iPhone/iPad via `ARWorldTrackingConfiguration.trackingObjects`.
- Spatial accessories open Apple Vision Pro's input model to custom hardware manufacturers, enabling physical game controllers and specialized professional tools that integrate seamlessly with the system.
- The hot-swap API (`provider.updateAccessories`) avoids session interruption when switching between accessories, a key quality-of-life improvement for multi-accessory apps.

---
_Source: WWDC26 Session 283 page (abstract, chapter summaries, code samples, and resource links)._
