# What's New in Nearby Interaction
**WWDC22 · Session 10008** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10008/)

_Platforms:_ iOS 16, watchOS 9

## Overview
Nearby Interaction leverages Apple's U1 Ultra Wideband chip for precise spatial awareness between Apple devices and compatible accessories. Two major additions arrive in iOS 16: ARKit-enhanced camera assistance mode and background accessory sessions.

Camera assistance integrates ARKit's device trajectory computation with UWB measurements to provide more consistent distance, direction, horizontal angle, and vertical direction estimates — enabling guide-to-object experiences similar to AirTag's Precision Finding. Background sessions allow Nearby Interaction with Bluetooth LE-paired accessories to continue running even when the app is backgrounded or the screen is locked, enabling hands-free use cases like automatically triggering actions when a user enters a room.

Third-party UWB-compatible development kits (previously in beta) are now generally available, and the accessory specification has been updated to include the new Nearby Interaction GATT service for background sessions.

## Key Topics

### ARKit-Enhanced Camera Assistance
A new `isCameraAssistanceEnabled` property on `NIConfiguration` subclasses activates a mode that uses ARKit's world-tracking to augment UWB measurements. Effectively expands the UWB field of view, making direction, horizontal angle, and vertical direction estimate available in more scenarios. Intended for guiding users to stationary nearby objects. Requires camera usage description in Info.plist.

If the app already uses ARKit, share the existing `ARSession` with the `NISession` via `NISession.setARSession(_:)` to run both concurrently. The shared `ARSession` must be configured with specific properties (gravity world alignment, no collaboration, no face tracking, nil initial world map).

### New Spatial Properties on NINearbyObject
- `horizontalAngle` — 1D azimuthal angle (radians) in the horizontal plane to the nearby object; nil when unavailable
- `verticalDirectionEstimate` — qualitative vertical relationship: `.same`, `.above`, `.below`, `.aboveOrBelow`, `.unknown`

### worldTransform Helper
`NISession.worldTransform(for:)` returns an ARKit coordinate-space transform for placing AR content at the nearby object's physical location. Returns nil until sufficient device motion (horizontal + vertical sweep) has been achieved.

### Algorithm Convergence Delegate
New `niSession(_:didUpdateAlgorithmConvergence:peer:)` delegate method provides an `NIAlgorithmConvergence` object explaining why spatial properties are unavailable and what user actions can help. Convergence reasons include insufficient total motion, insufficient horizontal/vertical sweep, and insufficient lighting.

### Accessory Background Sessions
Enables NISession with BLE-paired accessories to continue running while the app is backgrounded. The accessory receives UWB measurements in the background; the app does not receive `didUpdateNearbyObjects` callbacks until returning to foreground. Setup requires:
1. Bluetooth LE pairing (one-time user prompt)
2. New `NINearbyAccessoryConfiguration(accessoryData:bluetoothPeerIdentifier:)` initializer
3. Accessory must implement the new Nearby Interaction GATT service (single encrypted characteristic: Accessory Configuration Data)
4. `NearbyInteraction` entry in `UIBackgroundModes` Info.plist array

### Device Capabilities API
`NISession.isSupported` is now deprecated. Replaced by `NISession.deviceCapabilities` returning an `NIDeviceCapability` object with:
- `supportsPreciseDistanceMeasurement` — equivalent to former `isSupported`
- `supportsDirectionMeasurement`
- `supportsCameraAssistance`

## APIs & Frameworks

**NearbyInteraction**
- `NIConfiguration.isCameraAssistanceEnabled` **[NEW]** — enable ARKit-assisted mode
- `NINearbyAccessoryConfiguration` — existing class; new initializer:
  - `init(accessoryData:bluetoothPeerIdentifier:)` **[NEW]** — for background sessions with BLE peer ID
- `NISession.setARSession(_:)` **[NEW]** — share existing ARSession with NISession
- `NISession.worldTransform(for:)` **[NEW]** — returns ARKit world transform for nearby object
- `NISession.deviceCapabilities` **[NEW]** — replaces deprecated `isSupported`
- `NISession.isSupported` — **[DEPRECATED]** replaced by `deviceCapabilities`
- `NIDeviceCapability` **[NEW]** — describes device UWB capabilities
  - `supportsPreciseDistanceMeasurement`
  - `supportsDirectionMeasurement`
  - `supportsCameraAssistance`
- `NINearbyObject.horizontalAngle` **[NEW]** — 1D azimuthal angle (Float?, radians)
- `NINearbyObject.verticalDirectionEstimate` **[NEW]** — `NINearbyObject.VerticalDirectionEstimate` enum
- `NINearbyObject.VerticalDirectionEstimate` **[NEW]** — `.unknown`, `.same`, `.above`, `.below`, `.aboveOrBelow`
- `NIAlgorithmConvergence` **[NEW]** — convergence status object
- `NIAlgorithmConvergenceStatus` **[NEW]** — `.unknown`, `.converged`, `.notConverged([Reasons])`
- `NIAlgorithmConvergenceStatus.Reasons` **[NEW]** — `.insufficientMotion`, `.insufficientHorizontalSweep`, `.insufficientVerticalSweep`, `.insufficientLighting`
- `NISessionDelegate.session(_:didUpdateAlgorithmConvergence:peer:)` **[NEW]** — convergence update callback
- `NIError.invalidARConfiguration` **[NEW]** — error when shared ARSession config is incompatible

**CoreBluetooth** (used for background accessory sessions)
- `CBCentralManager` — peripheral scanning/connection
- `CBPeripheral.identifier` — used as `bluetoothPeerIdentifier` for NISession
- `UIBackgroundModes: NearbyInteraction` — required plist entry for background sessions

## Code Highlights

```swift
// Enable ARKit camera assistance
let config = NINearbyAccessoryConfiguration(accessoryData: data)
config.isCameraAssistanceEnabled = true
session.run(config)

// Background session with BLE-paired accessory
let config = NINearbyAccessoryConfiguration(
    accessoryData: accessoryUWBData,
    bluetoothPeerIdentifier: peripheral.identifier)
niSession.run(config)

// Check convergence and guide user
func session(_ session: NISession,
             didUpdateAlgorithmConvergence convergence: NIAlgorithmConvergence,
             peer: NINearbyObject?) {
    switch convergence.status {
    case .converged:
        showReadyUI()
    case .notConverged(let reasons):
        for reason in reasons {
            showGuidance(reason.localizedDescription)
        }
    default: break
    }
}

// Check device capabilities
let caps = NISession.deviceCapabilities
if caps.supportsCameraAssistance {
    // enable enhanced mode
}
```

## Takeaways

- Camera assistance with `isCameraAssistanceEnabled` expands the effective UWB field of view by combining ARKit world-tracking — enabling AirTag-style Precision Finding experiences in third-party apps.
- Background accessory sessions enable fully hands-free UWB interactions; the accessory processes measurements and triggers actions while the app is backgrounded.
- Use the new `NIAlgorithmConvergence` delegate to guide users through device-sweep motions needed to resolve horizontal angle, vertical direction, and world transform.
- Replace `NISession.isSupported` with `NISession.deviceCapabilities` to properly tailor experiences for devices with varying UWB capabilities (especially Apple Watch which supports distance but not direction).

---
_Source: WWDC22 Session 10008 page (abstract, chapter summaries, code samples, and resource links)._
