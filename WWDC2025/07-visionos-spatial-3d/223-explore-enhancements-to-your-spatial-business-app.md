# Explore Enhancements to Your Spatial Business App
**WWDC25 · Session 223** · [Watch](https://developer.apple.com/videos/play/wwdc2025/223/)

_Platforms:_ visionOS 26 (Enterprise APIs — in-house and custom B2B apps)

## Overview
This session presents the second generation of enterprise APIs for visionOS, organized around three pillars: streamlining development (wider API access, command-line object-tracking training, simplified license management), enhancing user experience (Window Follow Mode, shared coordinate spaces, content capture protection), and environment visualization (expanded stereo camera access, new Camera Region feature).

Several APIs previously requiring enterprise entitlements — UVC video and Neural Engine access — are now open to all developers. New enterprise-only features include `SharedCoordinateSpaceProvider` for custom collaborative experiences, `contentCaptureProtected` for data privacy, and `CameraRegionView` / `CameraRegionAnchor` for magnified real-world views.

## Key Topics

### Streamlined Development

**Wider API Access**
- **UVC device access** and **Neural Engine** — **[NOW OPEN]** no longer require an enterprise license; available to all developers.
- Object tracking model training — **[NEW]** available from the command line: `xcrun createml objecttracker -s my.usdz -o my.referenceobject`. Enables CI/CD integration without the CreateML GUI.

**Enterprise License Management**
- License files accessible directly in Apple Developer account; renewals pushed over-the-air automatically.
- **`VisionEntitlementServices`** framework — **[NEW]** check license validity and per-feature approval programmatically.
  - `EnterpriseLicenseDetails.shared.licenseStatus` — `.valid`, `.expired`, etc.
  - `EnterpriseLicenseDetails.shared.isApproved(for:)` — check specific entitlements (e.g., `.mainCameraAccess`, `.increasedPerformanceHeadroom`).

### User Experience Enhancements

**Window Follow Mode**
- **[NEW]** A user can designate a window to "follow" them as they move through space — the window translates with their position.
- Enabled via the `window-body-follow` licensed entitlement.
- Activated by the user: click-and-hold the window close control → "Start Follow Mode."

**Shared Coordinate Spaces (Enterprise ARKit)**
- **`SharedCoordinateSpaceProvider`** — **[NEW]** ARKit data provider for enterprise apps needing custom networking infrastructure for multi-user spatial experiences.
- Each participant runs `SharedCoordinateSpaceProvider` on their `ARKitSession`.
- `provider.nextCoordinateSpaceData` — pull API; returns `CoordinateSpaceData` to broadcast to others.
- `provider.push(data:)` — receives `CoordinateSpaceData` from other participants.
- `provider.eventUpdates` — `AsyncSequence` with `.connectedParticipantIdentifiers`, `.sharingEnabled`, `.sharingDisabled` events.
- Shared world anchors via `WorldAnchor(originFromAnchorTransform:sharedWithNearbyParticipants: true)`.
- For higher-level SharePlay-based shared spaces, see companion Session 318.

**Content Capture Protection**
- **`contentCaptureProtected()`** — **[NEW]** SwiftUI view modifier; requires `protected-content` entitlement.
- Applies to any `View` or RealityKit scene; the system automatically obscures the content in screen captures, recordings, mirroring, and SharePlay remote displays.
- Content remains fully visible to the wearer. Can be combined with Optic ID or corporate SSO authentication flows.

### Environment Visualization

**Expanded Camera Access**
- `CameraFrameProvider` (ARKit) — expanded to support individual left/right cameras or stereo pair (previously only left main camera). **[NEW]**
- Camera feeds now work in both Immersive Space and Shared Space environments. **[NEW]**

**Camera Regions**
- `CameraRegionView` (VisionKit SwiftUI) — **[NEW]** a window that displays a live stabilized/enhanced video feed of the real-world region directly behind the window.
  - `isContrastAndVibrancyEnhancementEnabled: true` — enable contrast and brightness optimization.
  - Optional pixel-processing closure: receive each `CVPixelBuffer`, perform custom analysis or modifications, and return the modified buffer (or `nil` to display unmodified).
  - Must reside in its own `WindowGroup` (window position drives region selection).
- `CameraRegionProvider` (ARKit) — **[NEW]** lower-level API for precise anchor-based region control.
- `CameraRegionAnchor` — defined by a `simd_float4x4` transform, width, height (meters), and enhancement mode (`.stabilization` or `.contrastAndVibrancyEnhancement`).
- `CameraRegionProvider.anchorUpdates(forID:)` — delivers `pixelBuffer` updates for the anchor.

## APIs & Frameworks

### ARKit (NEW)
- `SharedCoordinateSpaceProvider`
- `SharedCoordinateSpaceProvider.CoordinateSpaceData`
- `CameraRegionProvider`
- `CameraRegionAnchor`
- `CameraFrameProvider` — expanded stereo and Shared Space support

### VisionKit (NEW)
- `CameraRegionView` — SwiftUI view

### VisionEntitlementServices (NEW)
- `EnterpriseLicenseDetails.shared`
- `EnterpriseLicenseDetails.licenseStatus`
- `EnterpriseLicenseDetails.isApproved(for:)`

### SwiftUI (NEW modifier)
- `.contentCaptureProtected()` — requires `protected-content` entitlement

## Code Highlights

```swift
// Check enterprise license before using main camera
import VisionEntitlementServices
let license = EnterpriseLicenseDetails.shared
guard license.licenseStatus == .valid,
      license.isApproved(for: .mainCameraAccess) else { return }
```

```swift
// Shared Coordinate Space — push/pull
let provider = SharedCoordinateSpaceProvider()
// Receive from network → push to ARKit
provider.push(data: CoordinateSpaceData(data: receivedData)!)
// Pull from ARKit → send on network
if let data = provider.nextCoordinateSpaceData { transmit(data) }
```

```swift
// Camera Region with custom processing
CameraRegionView(isContrastAndVibrancyEnhancementEnabled: true) { result in
    guard case .success(let frame) = result else { return nil }
    cameraFeedDelivery.frameUpdate(pixelBuffer: frame.pixelBuffer)
    return nil  // return modified buffer to display custom processing result
}
```

```swift
// Protect sensitive content
SensitiveDataView()
    .contentCaptureProtected()
```

## Takeaways
- UVC and Neural Engine access no longer require enterprise entitlements — open these capabilities to broader audiences immediately.
- Use `VisionEntitlementServices` to gate enterprise-only features in your app rather than relying solely on capability checks at compile time.
- `CameraRegionView` is the fastest path to adding real-world magnification: two `WindowGroup` entries and a handful of lines of SwiftUI code.
- `contentCaptureProtected` is essential for enterprise apps that handle patient records, financial data, or proprietary designs — apply it at the view level and combine with authentication for defense-in-depth.

---
_Source: WWDC25 Session 223 page (abstract, chapters, full transcript, and code samples)._
