# Meet Nearby Interaction
**WWDC20 · Session 10668** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10668/)

_Platforms:_ iOS 14 (U1-equipped devices: iPhone 11 and later)

## Overview
The Nearby Interaction framework is new in iOS 14, providing apps access to the Apple U1 chip's Ultra-Wideband (UWB) hardware for real-time distance and direction measurements between nearby devices. Each framework session streams a continuous feed of `NINearbyObject` updates containing a distance in meters and a 3D direction unit vector. This enables spatially-aware interactions — similar to the AirDrop device-targeting experience introduced in iOS 13 — to be built into third-party apps.

The framework is session-based and privacy-first: users on both ends must grant explicit one-time permission before sessions can begin. Discovery tokens — randomly generated, session-scoped, time-limited identifiers — let devices find each other without exposing persistent device identifiers. Each app manages its own token exchange over its existing networking layer (e.g., MultipeerConnectivity, cloud relay).

## Key Topics

### Privacy and Permissions
- System presents a permission prompt to both users before any session can start; one-time permission that is revoked when the app exits
- Discovery tokens are randomly generated per session, expire with the session, and cannot be reused — no persistent device identifier is ever exposed

### Discovery Token Exchange
- Each `NISession` exposes a `discoveryToken: NIDiscoveryToken` property after creation
- `NIDiscoveryToken` conforms to `NSSecureCoding` — encode with `NSKeyedArchiver`, transport over any app networking layer (MultipeerConnectivity, CloudKit, etc.), decode with `NSKeyedUnarchiver` on the other end
- Exchange must be bidirectional: both devices send their token to the other before configuring the session

### Session Lifecycle
- Create one `NISession` per peer; a device can run multiple sessions simultaneously (one per peer)
- Set a delegate conforming to `NISessionDelegate` on each session
- Create an `NINearbyPeerConfiguration(peerToken:)` with the received token, then call `session.run(config)` to start streaming
- **Suspend/resume**: session suspends automatically when the app leaves the foreground; delegate receives `sessionWasSuspended(_:)` and later `sessionSuspensionEnded(_:)`; call `run` again (no need to re-exchange tokens) to resume
- **Invalidation**: `session(_:didInvalidateWithError:)` indicates the session cannot be reused; create a new session and re-exchange tokens

### NINearbyObject Updates
- Delegate method `session(_:didUpdate:)` receives an array of `NINearbyObject`
- Each object has three properties:
  - `discoveryToken: NIDiscoveryToken` — identifies which peer this update is for
  - `distance: Float?` — distance in meters; `nil` if unavailable
  - `direction: simd_float3?` — unit vector pointing toward the peer relative to the local device; `nil` if outside field of view

### Removal Reasons
- `session(_:didRemove:withReason:)` — called when the system stops tracking a peer
  - `.timeout` — no activity for a period (e.g., devices too far apart)
  - `.peerEnded` — other side explicitly invalidated its session (best-effort delivery)

### Hardware and Field of View
- Only available on U1-equipped devices (iPhone 11 and later); check `NISession.isSupported` before use
- Directional field of view approximates the Ultra Wide camera's cone out the back of the device
- Within the field of view: both `distance` and `direction` are produced
- Outside the field of view: `distance` may still be produced but `direction` is `nil`
- Best performance: both devices in portrait orientation and in clear line of sight of each other
- Physical occlusions (walls, people) reduce measurement availability

### Simulator Support
- Xcode provides native Nearby Interaction Simulator support — two Simulator windows can stream real distance and direction updates to the same app code without U1 hardware

## APIs & Frameworks

- **NearbyInteraction**
  - `NISession` **[NEW]** — the core session object; one per peer
  - `NISession.isSupported` **[NEW]** — class property; `Bool`; `false` on non-U1 devices
  - `NISession.discoveryToken: NIDiscoveryToken?` **[NEW]** — the local session's discovery token; set after session creation
  - `NISession.delegate: NISessionDelegate?` **[NEW]** — delegate for receiving updates and lifecycle events
  - `NISession.run(_:)` **[NEW]** — starts or resumes the session with the given configuration
  - `NISession.invalidate()` **[NEW]** — permanently invalidates the session
  - `NIDiscoveryToken` **[NEW]** — randomly generated, session-scoped device identifier; conforms to `NSSecureCoding`
  - `NINearbyPeerConfiguration` **[NEW]** — session configuration; initialized with `peerToken: NIDiscoveryToken`
  - `NINearbyObject` **[NEW]** — update object; properties: `discoveryToken`, `distance: Float?`, `direction: simd_float3?`
  - `NINearbyObject.RemovalReason` **[NEW]** — `.timeout` or `.peerEnded`
  - `NISessionDelegate` **[NEW]** — protocol with callbacks:
    - `session(_:didUpdate:)` — array of `NINearbyObject` with fresh measurements
    - `session(_:didRemove:withReason:)` — peer no longer tracked
    - `sessionWasSuspended(_:)` — session suspended (app backgrounded, etc.)
    - `sessionSuspensionEnded(_:)` — suspension lifted; call `run` to resume
    - `session(_:didInvalidateWithError:)` — session permanently invalidated

## Code Highlights

Setting up and running a session:
```swift
var niSession: NISession?

func prepareMySession() {
    guard NISession.isSupported else {
        print("Nearby Interaction is not available on this device.")
        return
    }
    niSession = NISession()
    niSession?.delegate = self
}

func sendDiscoveryTokenToMyPeer() {
    guard let myToken = niSession?.discoveryToken else { return }
    if let encodedToken = try? NSKeyedArchiver.archivedData(withRootObject: myToken,
                                                             requiringSecureCoding: true) {
        // Send encodedToken over your app's networking layer
    }
}

func runMySession(peerTokenData: Data) {
    guard let peerToken = try? NSKeyedUnarchiver.unarchivedObject(ofClass: NIDiscoveryToken.self,
                                                                   from: peerTokenData) else { return }
    let config = NINearbyPeerConfiguration(peerToken: peerToken)
    niSession?.run(config)
}
```

Handling updates in the delegate:
```swift
func session(_ session: NISession, didUpdate nearbyObjects: [NINearbyObject]) {
    for object in nearbyObjects {
        if let distance = object.distance {
            print("Distance: \(distance) m")
        }
        if let direction = object.direction {
            print("Direction: \(direction)")
        }
    }
}

func session(_ session: NISession, didRemove nearbyObjects: [NINearbyObject], withReason reason: NINearbyObject.RemovalReason) {
    if reason == .peerEnded {
        // Peer explicitly ended — clean up
    }
}

func sessionSuspensionEnded(_ session: NISession) {
    // Resume with the same config — no need to re-exchange tokens
    session.run(session.configuration!)
}
```

## Takeaways
- Always check `NISession.isSupported` and provide a fallback UX for non-U1 devices.
- `distance` and `direction` on `NINearbyObject` are both optional — direction is `nil` when the peer is outside the directional field of view; handle both cases explicitly.
- Maintain one `NISession` per peer; store them in a dictionary keyed by your app's peer identifier.
- Token exchange must be bidirectional and happen before calling `run`; use any existing app transport (MultipeerConnectivity, CloudKit, etc.) to shuttle the `NSSecureCoding`-encoded `NIDiscoveryToken`.
- Invalidated sessions cannot be reused — create a new session and re-exchange tokens; suspended sessions can resume with a second `run` call using the original configuration.

---
_Source: WWDC20 Session 10668 page (abstract, transcript, and code samples)._
