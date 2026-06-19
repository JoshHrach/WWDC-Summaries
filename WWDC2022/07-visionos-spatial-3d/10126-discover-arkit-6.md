# Discover ARKit 6
**WWDC22 · Session 10126** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10126/)

_Platforms:_ iOS 16, iPadOS 16

## Overview
ARKit 6 introduces a significant set of improvements to the augmented reality framework, focusing on higher resolution video, richer camera control, improved plane anchors, motion capture enhancements, and expanded geographic coverage for Location Anchors. The headline feature is 4K video mode — apps can now capture AR experiences at 3840×2160 at 30fps on supported hardware, skipping the traditional sensor binning step that previously capped video at HD resolution.

Beyond 4K, ARKit 6 adds a high-resolution background photo API for capturing native-resolution still frames during an active session (up to 12 megapixels on iPhone 13), HDR mode support, direct access to the underlying `AVCaptureDevice` for fine-grained camera control, and EXIF tag delivery per frame. Plane anchor behavior is also cleaned up with a decoupled geometry update model and a new `ARPlaneExtent.rotationOnYAxis` property.

## Key Topics

### 4K Video Mode
- New 4K format delivers 3840×2160 pixels at 30fps — achieved by bypassing the 2×2 binning step used in HD mode.
- Standard HD mode uses binning: 3840×2880 sensor area → 1920×1440 output, enabling up to 60fps.
- 4K mode skips binning and crops to 16:9 (3840×2160); aspect ratio means iPad users see zoomed-in crops in full-screen.
- `ARConfiguration.recommendedVideoFormatFor4KResolution` — convenience function, returns nil if unsupported **[NEW]**.
- Available on iPhone 11 and later, iPad Pro with M1 chip.
- Best for filmmaking, virtual production, video apps; games/high-refresh-rate apps should prefer 60fps HD.
- Do not retain `ARFrame` objects longer than needed — causes frame drops and tracking degradation.

### High-Resolution Background Photos
- Capture full-resolution still images (12MP on iPhone 13 Pro) while the AR session continues uninterrupted.
- `ARWorldTrackingConfiguration.recommendedVideoFormatForHighResolutionFrameCapturing` — check supported format **[NEW]**.
- `ARSession.captureHighResolutionFrame(completionHandler:)` — asynchronous, out-of-band capture **[NEW]**.
- Returns an `ARFrame` with full-resolution `capturedImage`; check for errors before using.
- Use cases: photography apps, photogrammetry (Object Capture workflows), guided capture UX.

### HDR Mode
- `ARVideoFormat.isVideoHDRSupported` — check if format supports HDR **[NEW]**.
- `ARConfiguration.videoHDRAllowed = true` — enable HDR **[NEW]**.
- Only non-binned (4K) video formats support HDR; has performance impact.
- HDR preserves detail in high-contrast scenes (bright skies, dark shadows).

### Direct AVCaptureDevice Access
- `ARWorldTrackingConfiguration.configurableCaptureDeviceForPrimaryCamera` — returns the underlying `AVCaptureDevice` **[NEW]**.
- Allows custom exposure, white balance, focus settings for creative looks.
- Caution: changes affect the image used for AR tracking — strong overexposure can degrade tracking quality.
- EXIF tags now delivered with every `ARFrame` **[NEW]** — useful for post-processing (white balance, exposure data).

### Plane Anchor Improvements
- Plane anchor rotation and geometry updates are now fully decoupled **[NEW behavior in iOS 16]**.
- Plane anchor orientation (`rotation`) no longer changes when plane geometry extends.
- New `ARPlaneExtent` class **[NEW]** contains all geometry: `width`, `height`, `center`, `rotationOnYAxis`.
- `ARPlaneExtent.rotationOnYAxis` — angle of plane rotation expressed separately from anchor transform.
- Set deployment target to iOS 16 to adopt new behavior.

### Motion Capture Improvements
- 2D skeleton: two new joints — left ear and right ear **[NEW]**.
- Improved overall pose detection accuracy.
- 3D skeleton (iPhone 12+, iPad Pro M1, iPad Air M1): less jitter, better temporal consistency, more stable occlusion handling.
- Set deployment target to iOS 16 to adopt improvements.

### Location Anchors — New Regions
- Previously available in select US cities and London.
- New regions added: Vancouver, Toronto, Montreal (Canada); Singapore; 7 metropolitan areas in Japan including Tokyo; Melbourne and Sydney (Australia).
- Later in 2022: Auckland (New Zealand), Tel Aviv (Israel), Paris (France).
- `ARGeoTrackingConfiguration.checkAvailability(at:completionHandler:)` — check if supported at a coordinate.

## APIs & Frameworks

### ARKit
- `ARConfiguration.recommendedVideoFormatFor4KResolution` **[NEW]**
- `ARConfiguration.videoHDRAllowed` **[NEW]**
- `ARVideoFormat.isVideoHDRSupported` **[NEW]**
- `ARWorldTrackingConfiguration.recommendedVideoFormatForHighResolutionFrameCapturing` **[NEW]**
- `ARWorldTrackingConfiguration.configurableCaptureDeviceForPrimaryCamera` **[NEW]**
- `ARSession.captureHighResolutionFrame(completionHandler:)` **[NEW]**
- `ARFrame.capturedImage` — pixel buffer of current frame
- `ARFrame.exifData` — EXIF metadata per frame **[NEW]**
- `ARPlaneExtent` — new class for plane geometry **[NEW]**
- `ARPlaneExtent.rotationOnYAxis` **[NEW]**
- `ARPlaneExtent.width`, `ARPlaneExtent.height`
- `ARPlaneAnchor.center` — center coordinate
- `ARGeoTrackingConfiguration.checkAvailability(at:completionHandler:)` — location anchor availability check
- `ARBodyAnchor` / body joint tracking — 2D skeleton now includes left/right ear joints **[NEW]**
- `ARConfiguration.videoFormat` — assign video format to configuration
- `ARSession.currentFrame.capturedImage` — access current frame for custom Metal renderers

### RealityKit
- Automatic scaling, cropping, and rendering of AR backdrop frames (existing)

## Code Highlights

```swift
// Enable 4K video mode
if let format4K = ARWorldTrackingConfiguration.recommendedVideoFormatFor4KResolution {
    config.videoFormat = format4K
}
session.run(config)

// Capture high-resolution background photo
session.captureHighResolutionFrame { (frame, error) in
    if let frame = frame {
        saveHiResImage(frame.capturedImage)
    }
}

// Enable HDR
if config.videoFormat.isVideoHDRSupported {
    config.videoHDRAllowed = true
}
session.run(config)

// Access underlying AVCaptureDevice
if let device = ARWorldTrackingConfiguration.configurableCaptureDeviceForPrimaryCamera {
    try device.lockForConfiguration()
    // configure custom exposure, white balance, etc.
    device.unlockForConfiguration()
}

// Create plane visualization using ARPlaneExtent
let planeEntity = ModelEntity(
    mesh: .generatePlane(width: planeExtent.width, depth: planeExtent.height),
    materials: [material])
planeEntity.transform = Transform(pitch: 0, yaw: planeExtent.rotationOnYAxis, roll: 0)
planeEntity.transform.translation = planeAnchor.center
```

## Takeaways
- ARKit 6's 4K mode unlocks professional-grade video for filmmaking and virtual production apps; HD 60fps remains better for games and high-interactivity experiences.
- The new high-resolution background photo API enables hybrid AR+photography apps and improves Object Capture workflows without requiring a separate `AVCaptureSession`.
- Plane anchor geometry updates are now fully decoupled from anchor rotation — use `ARPlaneExtent.rotationOnYAxis` to handle plane orientation in iOS 16.
- Motion capture gains two new ear joints and improved 3D skeleton stability on A14+ devices.

---
_Source: WWDC22 Session 10126 page (abstract, chapter summaries, code samples, and resource links)._
