# Run your iPad and iPhone apps in the Shared Space
**WWDC23 · Session 10090** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10090/)

_Platforms:_ visionOS 1, iOS 17, iPadOS 17

## Overview
visionOS is built on the same foundation as iOS and iPadOS. This means the majority of existing iPad and iPhone apps run on Apple Vision Pro without any code changes, appearing as floating windows in the Shared Space. This session covers what works automatically (built-in behaviors), what requires attention or adjustment (functional differences), and how to decide between keeping the "Designed for iPad" experience versus rebuilding natively for visionOS.

The key decision for developers is: does the iOS SDK path (Designed for iPad) meet the app's needs, or does the investment in the visionOS SDK unlock experiences important to the app? SpriteKit and storyboards are only available on the iOS SDK path; immersive spaces, volumes, Ornaments, and evolved ARKit/RealityKit are only available on the visionOS SDK path.

## Key Topics

### Built-in Behaviors (Free)
iOS/iPadOS apps run as windows in light mode style. The system:
- Prefers the iPad variant in landscape; falls back to iPhone portrait if iPad support is absent.
- Provides a rotation button above the window's top-right corner when the app supports multiple orientations.
- Scales windows via corner dragging while preserving aspect ratio (with min/max bounce feedback).
- Translates natural input (eye + hand pinch, direct touch, Bluetooth trackpad/gamepad) into familiar UIKit touch and pointer events.
- Uses system views (document picker, photo picker) matching the platform appearance.
- Forwards LocalAuthentication (Touch ID / Face ID) through Optic ID automatically.

### Functional Differences

**Orientation:** No device rotation concept — apps must specify a preferred default orientation via a new Info.plist key. Users can still rotate scenes with the system rotation button.

**Gestures:** Maximum two simultaneous touch inputs (one per hand). All system gestures expecting ≤2 touches work. Custom gesture recognizers may need updates for natural input expectations.

**ARKit:** Existing `ARView` / `ARSession` APIs do not work on visionOS. Apps using ARKit must rebuild against the visionOS SDK using new ARKit APIs. See "Evolve your ARKit app for spatial experiences" (10091).

**Camera:** Vision Pro has many cameras, but not all are available to apps. Always verify camera availability with capability checks before use.

**Location:** Similar to iPad; approximated via Wi-Fi or shared via iPhone.

**Look to Dictate:** A new input technique where users look at a microphone icon in search fields and speak to search. Disabled by default for iOS apps running on visionOS; must be explicitly opted in.

### Info.plist Keys
- `UIPreferredDefaultInterfaceOrientation` — **[NEW, visionOS-only]** specifies the preferred orientation for new scenes; ignored on other platforms.
- `UISupportedInterfaceOrientations` — system uses this to decide if a rotation button is needed.
- `UIRequiredDeviceCapabilities` — App Store Connect uses this to determine compatibility with Vision Pro; all compatible apps are automatically made available.

### Choosing the Right Path

**Designed for iPad (iOS SDK):**
- Apps run as windowed experiences with light mode iPad styling.
- Required if app uses SpriteKit or storyboards.
- All work done for iOS/iPadOS continues to benefit this experience.

**Designed for visionOS (visionOS SDK):**
- Unlock immersive spaces, volumes, Ornaments API, glass material backgrounds.
- Access evolved ARKit, RealityKit, and other visionOS-specific framework capabilities.
- App uses system look and feel (glass material, ornaments for tab bars/toolbars).

### Xcode Setup
With the visionOS SDK installed, Xcode automatically adds "visionOS Device (Designed for iPad)" to Supported Destinations for iOS SDK projects. Select it in the destination picker to build and run the iOS app on Vision Pro simulator or device.

## APIs & Frameworks

- **UIKit** — runs as-is on visionOS for Designed for iPad apps
  - `UISearchController` / `UISearchBar`
    - `UISearchBar.isLookToDictateEnabled: Bool` — enable Look to Dictate **[NEW]**
  - `UIPreferredDefaultInterfaceOrientation` — Info.plist key **[NEW, visionOS-only]**
  - `UISupportedInterfaceOrientations` — Info.plist key (existing)
  - `UIRequiredDeviceCapabilities` — Info.plist key (existing, visionOS compatibility)
- **SwiftUI**
  - `.searchable(text:)` — existing modifier
  - `.searchDictationBehavior(_:)` — new modifier to enable Look to Dictate **[NEW]**
    - `SearchDictationBehavior.inline(activation: .onLook)` — activates on gaze
- **LocalAuthentication** — Touch ID / Face ID automatically forwarded through Optic ID
- **ARKit** — existing `ARView` / `ARSession` not supported on visionOS; must rebuild with new APIs
- **SpriteKit** — available on iOS SDK (Designed for iPad) path only; not available on visionOS SDK
- **Storyboards** — available on iOS SDK path only
- **Ornaments API** — visionOS SDK only; anchors UI to window edges
- **Volumes** — visionOS SDK only; 3D content containers
- **Immersive Spaces** — visionOS SDK only
- **Optic ID** — authentication hardware on Vision Pro; LocalAuthentication forwards existing Touch/Face ID code to it automatically
- **Natural input** — eye + hand pinch, direct touch; maps to standard UIKit touch/pointer events
- **Bluetooth trackpad / game controller** — additional input methods supported

## Code Highlights

Enabling Look to Dictate in SwiftUI:
```swift
@State private var searchText = ""
var body: some View {
    NavigationStack {
        Text("Query: \(searchText)")
    }
    .searchable(text: $searchText)
    .searchDictationBehavior(.inline(activation: .onLook))
}
```

Enabling Look to Dictate in UIKit:
```swift
let searchController = UISearchController()
searchController.searchBar.isLookToDictateEnabled = true
```

Info.plist: set preferred default orientation for visionOS (does not affect iOS):
```xml
<key>UIPreferredDefaultInterfaceOrientation</key>
<string>UIInterfaceOrientationLandscapeRight</string>
```

## Takeaways

- Most iOS and iPadOS apps run on Vision Pro without any code changes; the system handles windowing, scaling, input translation, and authentication forwarding automatically.
- Review and update `UIRequiredDeviceCapabilities` in Info.plist to control App Store availability on Vision Pro; all compatible apps are automatically listed.
- Choose "Designed for iPad" (iOS SDK) if the app uses SpriteKit or storyboards, or if preserving the existing iOS experience is the goal; choose "Designed for visionOS" to unlock immersive spaces, volumes, Ornaments, evolved ARKit/RealityKit, and the glass material system look.
- Look to Dictate is off by default for iOS apps on visionOS — opt in explicitly using `.searchDictationBehavior()` or `isLookToDictateEnabled` where the experience makes sense.

---
_Source: WWDC23 Session 10090 page (abstract, chapter summaries, code samples, and resource links)._
