# Find your accessory with Bluetooth Channel Sounding
**WWDC26 · Session 369** · [Watch](https://developer.apple.com/videos/play/wwdc2026/369/)

_Platforms:_ iOS 27+, iPadOS 27+

## Overview
Bluetooth Channel Sounding is a new radio-level ranging technology in the Bluetooth 6.0 specification that measures the round-trip time of RF signals across multiple frequency channels, producing accurate sub-meter distance estimates. iOS 27 adds developer APIs exposing this capability through two frameworks: Core Bluetooth (distance only) and Nearby Interaction (distance plus direction, with camera-assisted angle estimation).

The session describes typical use cases — finding a lost luggage tag, locating a bike lock, guiding a user to a shared scooter — and walks through both API paths. Accessory-side hardware and firmware requirements are also covered: accessories must support the Bluetooth Channel Sounding specification, and developers should follow hardware design tips around antenna placement and power consumption to maximize ranging accuracy and battery life.

The two APIs serve different needs: Core Bluetooth's `CBChannelSoundingSessionConfiguration` is simpler and gives distance only; Nearby Interaction's `NINearbyAccessoryConfiguration` with `bluetoothChannelSoundingIdentifier` provides both distance and directional angle, and optionally fuses with camera data for improved direction accuracy.

## Key Topics

### Overview / Use Cases
Channel Sounding enables precise distance measurement (sub-meter accuracy) and, combined with Nearby Interaction, directional guidance. Example applications: item finders, access control (unlock when nearby), parking spot or bike-share location.

### Core Bluetooth API
A two-step process: check `CBCentralManager.supportsFeatures(.channelSounding)`, create a `CBChannelSoundingSessionConfiguration` with role `.initiator`, call `peripheral.startChannelSoundingSession(_:)`. Distance results arrive in the `CBPeripheralDelegate` method `peripheral(_:didReceive:error:)`. Cancel with `peripheral.cancelChannelSoundingSession(_:)`.

### Nearby Interaction API
Build on top of Core Bluetooth by using `NINearbyAccessoryConfiguration(bluetoothChannelSoundingIdentifier:previousChannelSoundingIdentifier:)` and running an `NISession`. Provides both `distance` and `horizontalAngle` on `NINearbyObject`. Enable camera assistance for improved direction with `config.isCameraAssistanceEnabled = true`. Improve direction accuracy by reporting whether the accessory is moving via `session.updateMotionState(_:forObjectWithToken:)`.

### Hardware Tips
Accessories must implement the Bluetooth Channel Sounding spec. Antenna design, placement, and RF shielding affect ranging accuracy. Power management is critical: do not run continuous Channel Sounding sessions; use the results to trigger UI updates and suspend the session when not needed.

## APIs & Frameworks

### Core Bluetooth
- `CBCentralManager.supportsFeatures(_:)` **[NEW in iOS 27]** — check device capability; pass `.channelSounding`
- `CBCentralManager.CBFeature.channelSounding` **[NEW]** — feature flag
- `CBChannelSoundingSessionConfiguration` **[NEW]** — configuration for a channel sounding session; `init(role:)`
- `CBChannelSoundingSessionConfiguration.Role.initiator` **[NEW]** — iPhone acts as the initiator
- `CBPeripheral.startChannelSoundingSession(_:)` **[NEW]** — begins ranging; peripheral must be connected
- `CBPeripheral.cancelChannelSoundingSession(_:)` **[NEW]** — stops the session
- `CBPeripheralDelegate.peripheral(_:didReceive:error:)` **[NEW]** — delivers `CBChannelSoundingProcedureResults?`
- `CBChannelSoundingProcedureResults` **[NEW]** — contains `distance: Double?` (meters)
- `CBPeripheralDelegate.peripheral(_:didCompleteChannelSoundingSession:)` **[NEW]** — session ended callback

### Nearby Interaction
- `NISession.deviceCapabilities.supportsBluetoothChannelSounding` **[NEW]** — Bool; check before using NI Channel Sounding path
- `NISession.deviceCapabilities.supportsCameraAssistance` — existing capability check
- `NINearbyAccessoryConfiguration(bluetoothChannelSoundingIdentifier:previousChannelSoundingIdentifier:)` **[NEW]** — creates configuration from a `CBPeripheral.identifier`
- `NINearbyAccessoryConfiguration.isCameraAssistanceEnabled` — Bool; enables direction via camera fusion
- `NISession.run(_:)` — start the session with the accessory configuration
- `NISession.updateMotionState(_:forObjectWithToken:)` **[NEW]** — improves direction accuracy when accessory is moving or stationary
- `NIMotionActivityState` **[NEW]** — `.moving` / `.stationary`
- `NISessionDelegate.session(_:didUpdate:)` — delivers `[NINearbyObject]`
- `NINearbyObject.distance` — optional Float (meters)
- `NINearbyObject.horizontalAngle` — optional Float (radians, direction toward accessory)
- `NINearbyObject.discoveryToken` — used to correlate objects across delegate calls

### AccessorySetupKit
- Referenced for accessory pairing and setup; not the ranging API itself

## Code Highlights

Core Bluetooth distance-only ranging:
```swift
// Check support
guard CBCentralManager.supportsFeatures(.channelSounding) else { return }
// Start session
let config = CBChannelSoundingSessionConfiguration(role: .initiator)
peripheral.startChannelSoundingSession(config)

// Receive results
func peripheral(_ peripheral: CBPeripheral,
                didReceive results: CBChannelSoundingProcedureResults?, error: Error?) {
    let distance = results?.distance  // meters
}
```

Nearby Interaction distance + direction:
```swift
guard NISession.deviceCapabilities.supportsBluetoothChannelSounding else { return }
let config = NINearbyAccessoryConfiguration(
    bluetoothChannelSoundingIdentifier: peripheral.identifier,
    previousChannelSoundingIdentifier: nil)
if NISession.deviceCapabilities.supportsCameraAssistance {
    config.isCameraAssistanceEnabled = true
}
let session = NISession(); session.delegate = self; session.run(config)
```

## Takeaways
- iOS 27 introduces two API paths: Core Bluetooth for simple distance, Nearby Interaction for distance + direction with camera fusion.
- Both paths require Bluetooth Channel Sounding–capable accessories; check `supportsFeatures(.channelSounding)` / `supportsBluetoothChannelSounding` before starting sessions.
- Report accessory motion state to Nearby Interaction for better directional accuracy.
- Manage power carefully: run sessions only when the UI is actively guiding the user.

---
_Source: WWDC26 Session 369 page (abstract, chapter summaries, code samples, and resource links)._
