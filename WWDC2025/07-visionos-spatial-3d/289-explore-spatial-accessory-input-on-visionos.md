# Explore spatial accessory input on visionOS

**Session ID:** 289  
**WWDC Year:** 2025  
**Folder:** `07-visionos-spatial-3d`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/289/

---

## Overview

This session introduces the new spatial accessory input APIs in visionOS 26, enabling apps to receive input from physical spatial controllers — specifically the PlayStation VR2 Sense controller and the Logitech Muse stylus. It walks through how visionOS discovers and connects accessories via Bluetooth, how to enumerate available input devices, read button and thumbstick state, and use the new `SpatialAccessoryInputProvider` to integrate accessory input alongside existing visionOS input modalities such as hand tracking and eye gaze. The session also covers ARKit and RealityKit integration for mapping controller pose into 3D scene space.

---

## Key Topics

- New spatial accessories supported in visionOS 26: PS VR2 Sense controller and Logitech Muse stylus
- Connecting accessories via Bluetooth; pairing UI provided by the system
- Enumerating connected spatial accessories with `SpatialAccessoryInputProvider`
- Reading button events, thumbstick axes, and trigger values
- Mapping accessory 6DoF pose into RealityKit scene coordinates
- Handling connect/disconnect lifecycle
- Combining accessory input with existing hand and eye input
- Designing accessible interactions that work with and without a spatial accessory

---

## APIs & Frameworks

- **SpatialAccessoryInputProvider** – **[NEW]** (visionOS 26) Main entry point; `AsyncStream<SpatialAccessoryEvent>` source that delivers button, axis, and pose events from connected spatial accessories.
- **`SpatialAccessoryEvent`** – **[NEW]** Enum with cases: `.buttonChanged(accessory:button:isPressed:)`, `.axisChanged(accessory:axis:value:)`, `.poseChanged(accessory:pose:)`, `.accessoryConnected(_:)`, `.accessoryDisconnected(_:)`.
- **`SpatialAccessory`** – **[NEW]** Value type representing a connected accessory; properties: `id`, `name`, `kind` (`.senseController`, `.musStylus`).
- **`SpatialAccessoryButton`** – **[NEW]** Enum of button identifiers: `.primary`, `.secondary`, `.trigger`, `.grip`, `.thumbstickPress`, `.dpad(direction:)`.
- **`SpatialAccessoryAxis`** – **[NEW]** Enum of axis identifiers: `.thumbstickX`, `.thumbstickY`, `.triggerAnalog`.
- **`SpatialAccessoryPose`** – **[NEW]** Struct wrapping a `simd_float4x4` world-space transform for the accessory's tracked position and orientation.
- **ARKit `WorldTrackingProvider`** – Used alongside `SpatialAccessoryInputProvider` to resolve accessory pose into scene-anchored coordinates.
- **RealityKit `AnchorEntity`** – Attach a virtual model to a `SpatialAccessoryPose` transform to render a controller representation in the scene.
- **`ARKitSession`** – Start `WorldTrackingProvider` and `SpatialAccessoryInputProvider` in the same session; both are `DataProvider` conformances. **[NEW]** `SpatialAccessoryInputProvider` is a new `DataProvider`.
- **SwiftUI `.simultaneousGesture`** – Recommended pattern for layering accessory button events over existing SwiftUI window gestures.

---

## Code Highlights

Starting the input provider and consuming events:
```swift
import ARKit
import RealityKit

let session = ARKitSession()
let accessoryProvider = SpatialAccessoryInputProvider()
let worldTracking = WorldTrackingProvider()

try await session.run([worldTracking, accessoryProvider])

for await event in accessoryProvider.events {
    switch event {
    case .buttonChanged(let accessory, let button, let isPressed):
        if button == .trigger && isPressed {
            handleTriggerPress(for: accessory)
        }
    case .poseChanged(let accessory, let pose):
        updateControllerEntity(for: accessory, transform: pose.transform)
    case .accessoryConnected(let accessory):
        print("Connected: \(accessory.name)")
    default:
        break
    }
}
```

Rendering a controller mesh at the tracked pose:
```swift
func updateControllerEntity(for accessory: SpatialAccessory, transform: simd_float4x4) {
    controllerEntity.transform = Transform(matrix: transform)
}
```

---

## Takeaways

- visionOS 26 expands input beyond hand tracking and eye gaze to physical 6DoF controllers via the new `SpatialAccessoryInputProvider`.
- The PS VR2 Sense controller and Logitech Muse are the first supported accessories; the API is designed to accommodate future devices.
- System-provided Bluetooth pairing UI handles discovery, so apps do not need custom pairing flows.
- Pose events use the same coordinate space as ARKit world tracking, making it straightforward to anchor virtual objects to controller position.
- Accessibility best practice: always provide an equivalent interaction path using hand gestures or eye gaze for users who do not have a spatial accessory.
- `SpatialAccessoryInputProvider` is a DataProvider — run it in an `ARKitSession` alongside other providers, not in a separate session.
