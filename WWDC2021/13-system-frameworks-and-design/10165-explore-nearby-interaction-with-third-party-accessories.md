# Explore Nearby Interaction with Third-Party Accessories
**WWDC21 · Session 10165** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10165/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
This session covers two major updates to the Nearby Interaction framework in iOS 15: a redesigned persistent user permission model replacing the previous one-time-per-app-launch prompt, and new APIs enabling iPhone (U1 chip) to interact with compatible Ultra-Wideband (UWB) third-party accessories.

The third-party accessory support is built on an industry-standard specification co-developed with chipset manufacturers. Developers can use MFi-certified UWB development kits as hardware starting points and the new `NINearbyAccessoryConfiguration` API to run sessions that provide distance and direction measurements between an iPhone and a custom UWB-equipped accessory.

## Key Topics
- **New Permission Model (iOS 15):** The Nearby Interaction permission prompt now grants persistent "while in use" permission when the user taps "OK." The prompt appears only once (first session run), after which the app appears in Settings where users can revoke or re-grant access. Previous model was one-time-per-app-launch with only "Allow Once" / "Don't Allow" options. Apps are invalidated with a permission error if access is revoked; guide users to Settings accordingly.
- **Third-Party Accessory Support (NEW):** New `NINearbyAccessoryConfiguration` type (subclass of `NIConfiguration`) enables UWB sessions with third-party accessories. Requires U1-equipped iPhone (iPhone 11+). Based on published Nearby Interaction Accessory Protocol Specification (for chipset/module manufacturers) and a separate accessory specification for accessory manufacturers.
- **Configuration Data Exchange Protocol:** Bidirectional configuration data must be exchanged over an app-controlled data channel (Bluetooth, local network, custom protocol):
  1. Accessory generates Accessory Configuration Data and sends to app
  2. App creates `NINearbyAccessoryConfiguration(data:)`, calls `session.run(configuration:)`
  3. Framework generates Shareable Configuration Data and delivers via `session(_:didGenerateShareableConfigurationData:for:)` delegate callback
  4. App sends Shareable Configuration Data to accessory over data channel
  5. Accessory passes data to UWB hardware chip; ranging begins
- **Timeout Handling:** If Shareable Configuration Data is not delivered to the accessory quickly enough, the session times out with `.timeout` reason in `session(_:didRemove:reason:)`. Retry by calling `session.run(configuration:)` again with the cached configuration (valid as long as the accessory session has not been terminated on the accessory side).
- **Multiple Accessories:** Create and run a separate `NISession` per accessory. The `didGenerateShareableConfigurationData` callback identifies which accessory's data to send via the `nearbyObject` parameter.
- **Distance-Based Zone Logic:** `NearbyObject.distance` is provided in meters. Apply smoothing (e.g., hysteresis or moving average) before using for zone boundary logic to guard against jitter and boundary oscillation.

## APIs & Frameworks

**NearbyInteraction**
- `NISession` – Main session object (existing)
  - `session.run(_ configuration: NIConfiguration)` – Start session (existing)
- `NIConfiguration` – Base class (existing)
- `NINearbyAccessoryConfiguration` **[NEW]** – Configuration for third-party UWB accessory sessions
  - `NINearbyAccessoryConfiguration(data: Data)` **[NEW]** – Init with Accessory Configuration Data; throws if data is invalid format
  - `NINearbyAccessoryConfiguration.accessoryDiscoveryToken: NIDiscoveryToken` **[NEW]** – Automatically populated token for the accessory
- `NINearbyPeerConfiguration` – iPhone-to-iPhone configuration (existing, for comparison)
- `NISessionDelegate` – Delegate protocol
  - `session(_:didUpdate:)` – Distance/direction updates (existing)
  - `session(_:didRemove:reason:)` – Session ended, reason includes `.timeout` **[NEW enum case]**
  - `session(_:didGenerateShareableConfigurationData:for:)` **[NEW]** – Delivers Shareable Configuration Data to send to the accessory
  - `session(_:didInvalidateWithError:)` – Session invalidated (includes permission errors)
- `NearbyObject.distance: Float?` – Distance to nearby object in meters (existing)
- `NearbyObject.direction: simd_float3?` – Direction unit vector (existing, when supported)
- `NINearbyObjectRemovalReason` – `.timeout` **[NEW]** – Session timed out waiting for configuration data exchange

**Info.plist Key**
- `NSNearbyInteractionUsageDescription` – Required; shown in permission prompt; should clearly describe the feature requiring Nearby Interaction

**MFi / Accessory Specification**
- Nearby Interaction Accessory Protocol Specification – For UWB chipset/module manufacturers
- Nearby Interaction Accessory Manufacturer Specification – For accessory hardware developers
- Development kits from certified UWB chipset technology providers compatible with U1

## Code Highlights
Creating a NINearbyAccessoryConfiguration from received data:
```swift
func setupAccessory(configData: Data, name: String) {
    do {
        let config = try NINearbyAccessoryConfiguration(data: configData)
        // Cache token + name for correlating future NearbyObject updates
        discoveryTokenToName[config.accessoryDiscoveryToken] = name
        session = NISession()
        session.delegate = self
        session.run(config)
    } catch {
        print("Invalid accessory configuration data: \(error)")
    }
}
```

Sending Shareable Configuration Data back to the accessory:
```swift
func session(_ session: NISession,
             didGenerateShareableConfigurationData data: Data,
             for object: NINearbyObject) {
    // Route data back to the correct accessory over your data channel
    let connection = getConnection(for: object)
    connection.send(data)
}
```

Handling session timeout and retrying:
```swift
func session(_ session: NISession,
             didRemove nearbyObjects: [NINearbyObject],
             reason: NINearbyObjectRemovalReason) {
    if reason == .timeout, shouldRetry(for: nearbyObjects.first) {
        // Retry with the same cached configuration
        session.run(cachedConfiguration)
    }
}
```

Zone detection with distance smoothing:
```swift
func session(_ session: NISession, didUpdate nearbyObjects: [NINearbyObject]) {
    guard let object = nearbyObjects.first,
          let rawDistance = object.distance else { return }
    let smoothed = getSmoothedDistance(rawDistance)
    if smoothed < 1.5 {
        enableFunctionalityB()
    } else if smoothed < 3.0 {
        enableFunctionalityA()
    }
}
```

## Takeaways
- The persistent "while in use" permission model means the prompt shows only once; make sure the first `NISession.run()` call coincides with a clear user intent so the prompt context is obvious.
- The Shareable Configuration Data must be sent to the accessory as fast as possible after the `didGenerateShareableConfigurationData` callback—session timeout occurs if the exchange is too slow.
- Cache `NINearbyAccessoryConfiguration.accessoryDiscoveryToken` alongside the accessory's display name immediately after creating the configuration so that `NearbyObject` updates can be mapped back to a named device.
- Apply distance smoothing before acting on zone boundaries; raw UWB distance values can fluctuate rapidly at boundary edges.

---
_Source: WWDC21 Session 10165 page (abstract, transcript, and resource links)._
