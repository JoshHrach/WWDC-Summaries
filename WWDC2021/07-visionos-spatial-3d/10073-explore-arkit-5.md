# Explore ARKit 5
**WWDC21 · Session 10073** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10073/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
ARKit 5 brings a collection of meaningful improvements across location anchors, App Clip Code tracking, face tracking, and motion capture. The session covers expanded geographic support for Location Anchors (now including London—first city outside the US), a new `.geoTracking` coaching overlay goal for consistent onboarding, App Clip Code detection and tracking as a new `ARAnchor` type, ultra-wide front-camera face tracking on the new iPad Pro, and improved accuracy and range for body motion capture on A14 devices.

The App Clip Code tracking section is particularly notable: a new `ARAppClipCodeAnchor` type provides a URL, decoding state, and radius, enabling AR experiences that are discoverable through physical printed codes and can position virtual content with high precision at the code's location.

## Key Topics
- **Location Anchors – Expanded Regions:** Support now covers 25+ US cities plus London (first international market). Documentation at `ARGeoTrackingConfiguration` always reflects current regions. Use `ARGeoTrackingConfiguration.checkAvailability` before running.
- **GeoTracking Coaching Overlay (NEW):** New `.geoTracking` goal for `ARCoachingOverlayView`, showing an animated guide to point the device at building facades for fast localization.
- **Location Anchor Best Practices:** Use Reality Composer for ARKit session recording and Xcode replay for development without going outdoors. Place content floating in air rather than precisely overlapping structures to reduce placement difficulty. Use `CLLocation.distance(from:)` to add anchors only when within 50 m.
- **App Clip Code Tracking (NEW):** `ARWorldTrackingConfiguration.appClipCodeTrackingEnabled = true` enables `ARAppClipCodeAnchor` detection. Requires A12+. Three URL decoding states: `.decoding`, `.decoded`, `.failed`. Show placeholder UI while decoding, error feedback on failure, correct content once decoded. Can combine with image tracking for content placement offset from the code.
- **Face Tracking – Ultra-Wide Camera (NEW):** New iPad Pro supports an ultra-wide front-facing camera. Opt in by iterating `ARFaceTrackingConfiguration.supportedVideoFormats` and selecting `captureDeviceType == .builtInUltraWideCamera`. Note: no `capturedDepthData` buffer available on ARFrame when using ultra-wide format.
- **Motion Capture Improvements:** On A14 Bionic (iPhone 12+) and iOS 15, motion capture gains wider range of body poses, more accurate rotations, tracking from greater distances, and larger range of limb movement. No code changes required—all existing apps benefit automatically.

## APIs & Frameworks

**ARKit**
- `ARGeoTrackingConfiguration` – Location anchor world tracking (existing)
  - `ARGeoTrackingConfiguration.isSupported` – Device support check
  - `ARGeoTrackingConfiguration.checkAvailability(completionHandler:)` – Location availability check
- `ARGeoAnchor` – Geographic anchor with `CLLocationCoordinate2D` and optional altitude
- `ARGeoTrackingStatus` – Tracking state (`.localized`, `.localizing`, etc.)
- `ARCoachingOverlayView.goal = .geoTracking` **[NEW]** – New coaching overlay goal
- `ARAppClipCodeAnchor` **[NEW]** – New anchor type for App Clip Code detection
  - `ARAppClipCodeAnchor.url` – Decoded URL (available once `.decoded`)
  - `ARAppClipCodeAnchor.urlDecodingState` **[NEW]** – `.decoding`, `.decoded`, `.failed`
  - `ARAppClipCodeAnchor.radius` – Physical radius of the code in meters
  - `ARAppClipCodeAnchor.isTracked` – Whether currently tracked
- `ARWorldTrackingConfiguration.appClipCodeTrackingEnabled` **[NEW]** – Enables App Clip Code detection
- `ARWorldTrackingConfiguration.supportsAppClipCodeTracking` **[NEW]** – Device capability check
- `ARFaceTrackingConfiguration.supportedVideoFormats` – List of supported video formats
  - `AVCaptureDevice.DeviceType.builtInUltraWideCamera` – Ultra-wide camera device type **[NEW for face tracking]**
  - `ARVideoFormat.captureDeviceType` – Type of capture device for a video format
- Body motion capture – Automatic improvements on A14 Bionic + iOS 15; no API changes

**ARView / RealityKit**
- `ARView.raycast(from:allowing:alignment:)` – Hit-testing for content placement
- `AnchorEntity(anchor:)` – Attach entities to `ARAppClipCodeAnchor`

**App Clips**
- `ARAppClipCodeAnchor` integrates with App Clip URL routing

**CoreLocation**
- `CLLocationCoordinate2D` – Used in `ARGeoAnchor`
- `CLLocation.distance(from:)` – Computes distance in meters between two geographic points

## Code Highlights
Setting up geo tracking with coaching overlay:
```swift
let coachingOverlay = ARCoachingOverlayView()
coachingOverlay.session = arView.session
coachingOverlay.goal = .geoTracking
arView.addSubview(coachingOverlay)
```

Enabling and handling App Clip Code tracking:
```swift
guard ARWorldTrackingConfiguration.supportsAppClipCodeTracking else { return }
let config = ARWorldTrackingConfiguration()
config.appClipCodeTrackingEnabled = true
arSession.run(config)

// In didUpdate anchors:
guard let codeAnchor = anchor as? ARAppClipCodeAnchor, codeAnchor.isTracked else { return }
switch codeAnchor.urlDecodingState {
case .decoding: displayPlaceholder(over: codeAnchor)
case .failed:   displayError(over: codeAnchor)
case .decoded:  showContent(for: codeAnchor.url!, radius: codeAnchor.radius)
}
```

Opting into ultra-wide face tracking:
```swift
let config = ARFaceTrackingConfiguration()
for format in ARFaceTrackingConfiguration.supportedVideoFormats {
    if format.captureDeviceType == .builtInUltraWideCamera {
        config.videoFormat = format; break
    }
}
session.run(config)
```

## Takeaways
- The new `.geoTracking` coaching overlay goal provides a consistent, learnable onboarding UI for location anchor apps, especially important now that Maps itself uses the same API.
- `ARAppClipCodeAnchor` closes the loop between physical discovery (scan a printed code) and precise AR content placement, with a three-state decoding model that supports progressive UX feedback.
- Ultra-wide face tracking on iPad Pro requires an explicit opt-in and loses depth buffer access—design face tracking apps to degrade gracefully if `capturedDepthData` is nil.
- Motion capture accuracy and range improvements on A14 are automatic and require no code changes; all apps on iOS 15 benefit immediately.

---
_Source: WWDC21 Session 10073 page (abstract, chapter summaries, code samples, and resource links)._
