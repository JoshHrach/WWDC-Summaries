# AVCapturePhotoOutput - Beyond the Basics
**WWDC16 · Session 511** · [Watch](https://developer.apple.com/videos/play/wwdc2016/511/)

_Platforms:_ iOS 10

## Overview
This Chalk Talk is a companion addendum to Session 501 (Advances in iOS Photography) and covers three topics that were not addressed in the main session. It begins with Scene Monitoring, which allows apps to present real-time UI feedback about flash and still image stabilization suitability before the user takes a photo. It then covers Resource Preparation and Reclamation, explaining how `AVCapturePhotoOutput` supports on-demand allocation of capture resources to avoid both over-preparation (wasted memory, slower preview start) and under-preparation (missed shots due to runtime allocation). Finally, it covers the camera privacy policy changes mandatory for iOS 10.

## Key Topics

### Scene Monitoring
- `AVCapturePhotoOutput.photoSettingsForSceneMonitoring` **[NEW]** — assign a configured `AVCapturePhotoSettings` object to drive continuous scene evaluation. Set flash mode and/or `isAutoStillImageStabilizationEnabled` to indicate which features to evaluate.
- `AVCapturePhotoOutput.isFlashScene` **[NEW, KVO-observable]** — `true` when the current scene is dark enough to warrant flash.
- `AVCapturePhotoOutput.isStillImageStabilizationScene` **[NEW, KVO-observable]** — `true` when the current scene is suitable for SIS.
- Flash and SIS ranges overlap: in overlapping light levels, SIS takes precedence over flash because SIS yields better image quality. The scene monitoring properties reflect this priority.
- Default value is `nil` (no monitoring); properties return `false` indefinitely until scene monitoring settings are configured.
- Recommendation: monitor only the features your capture UI actually exposes to avoid confusing the user.

### Resource Preparation and Reclamation
- `AVCapturePhotoOutput` receives data only on demand (unlike streaming outputs), giving it downtime to prepare/reclaim resources without a disruptive pipeline rebuild.
- Different capture types require different resource levels: JPEG (minimal) < BGRA < Flash < SIS (multiple buffers) < RAW (very large buffers) < RAW+JPEG < bracketed combinations.
- `setPreparedPhotoSettingsArray(_:completionHandler:)` **[NEW]** — pass an array of `AVCapturePhotoSettings` representing every capture type you plan to use. The output prepares resources for all types and reclaims any unneeded ones. Pass an empty array to reclaim everything.
- `preparedPhotoSettingsArray` **[NEW]** — read-only property reflecting the last set preparation array. Default is a single default `AVCapturePhotoSettings` (JPEG + AutoSIS).
- Settings are sticky: persists across `startRunning`/`stopRunning` and `beginConfiguration`/`commitConfiguration`.
- Participates in `beginConfiguration`/`commitConfiguration` deferred semantics: preparation only occurs at `commitConfiguration`.
- Call before `startRunning` to ensure the session is ready to capture as soon as preview starts.
- If `setPreparedPhotoSettingsArray` is called while stopped, the completion handler fires after `startRunning`. A second call while stopped cancels the first (completion handler fires with `prepared: false`).
- Features that cannot be prepared on-demand (must be set before `startRunning`): `isHighResolutionCaptureEnabled`, `isLivePhotoCaptureEnabled`, `automaticallyTrimsLivePhotoCapturesToSupportedLength`.
- Re-prepare only when UI changes (e.g., user toggles RAW or bracketed capture), not on every shot.

### Camera Privacy Policy Changes (iOS 10)
- Apps linked against iOS 10 SDK **must** provide reason strings in `Info.plist` for all sensitive data access, or access will be denied.
- `NSCameraUsageDescription` **[NEW requirement]** — reason string for camera access.
- `NSMicrophoneUsageDescription` **[NEW requirement]** — reason string for microphone access (required when adding a microphone input for Live Photo audio).
- `NSPhotoLibraryUsageDescription` **[NEW requirement]** — reason string for photo library access; clarify whether reading, writing, or both.
- Xcode lists all available privacy description keys for all sensitive data categories.

## APIs & Frameworks

- **AVFoundation**
- `AVCapturePhotoOutput.photoSettingsForSceneMonitoring` **[NEW]** — nullable `AVCapturePhotoSettings` driving scene evaluation
- `AVCapturePhotoOutput.isFlashScene` **[NEW, KVO]** — current scene flash suitability
- `AVCapturePhotoOutput.isStillImageStabilizationScene` **[NEW, KVO]** — current scene SIS suitability
- `AVCapturePhotoOutput.setPreparedPhotoSettingsArray(_:completionHandler:)` **[NEW]** — on-demand resource prep/reclaim
- `AVCapturePhotoOutput.preparedPhotoSettingsArray` **[NEW]** — current prepared settings (read-only)
- `AVCapturePhotoOutput.isHighResolutionCaptureEnabled` — must be set before `startRunning`; enables >streaming-resolution stills
- `AVCapturePhotoOutput.isLivePhotoCaptureEnabled` — must be set before `startRunning`
- `AVCapturePhotoOutput.automaticallyTrimsLivePhotoCapturesToSupportedLength` — opt-out for full-duration untrimmed Live Photos; must be set before `startRunning`
- `AVCapturePhotoSettings.flashMode` — `.auto`, `.on`, `.off`
- `AVCapturePhotoSettings.isAutoStillImageStabilizationEnabled`
- `AVCaptureSession.startRunning()` / `stopRunning()`
- `AVCaptureSession.beginConfiguration()` / `commitConfiguration()`
- **Privacy keys (Info.plist)**
  - `NSCameraUsageDescription` **[NEW requirement in iOS 10]**
  - `NSMicrophoneUsageDescription` **[NEW requirement in iOS 10]**
  - `NSPhotoLibraryUsageDescription` **[NEW requirement in iOS 10]**

## Code Highlights

Setting up scene monitoring with KVO:
```swift
let monitorSettings = AVCapturePhotoSettings()
monitorSettings.flashMode = .auto
monitorSettings.isAutoStillImageStabilizationEnabled = true
photoOutput.photoSettingsForSceneMonitoring = monitorSettings

photoOutput.addObserver(self, forKeyPath: "isFlashScene", options: .new, context: nil)
photoOutput.addObserver(self, forKeyPath: "isStillImageStabilizationScene", options: .new, context: nil)
```

Preparing for multiple capture types:
```swift
let jpegSettings = AVCapturePhotoSettings()
let rawSettings = AVCapturePhotoSettings(rawPixelFormatType: rawFormat)
let bracketSettings = AVCapturePhotoBracketSettings(rawPixelFormatType: 0,
    processedFormat: nil, bracketedSettings: exposureBracket)

photoOutput.setPreparedPhotoSettingsArray([jpegSettings, rawSettings, bracketSettings]) { prepared, error in
    if prepared { print("Ready to capture") }
}
```

## Takeaways
- Use `photoSettingsForSceneMonitoring` and KVO on `isFlashScene`/`isStillImageStabilizationScene` to drive flash/SIS UI badges without taking a photo first; configure monitoring to match exactly what your capture UI offers.
- Call `setPreparedPhotoSettingsArray` before `startRunning` with every capture type your app supports to avoid runtime allocation delays on the first shot.
- Re-prepare only when the user changes capture mode (toggling RAW, bracketed, etc.); the settings are sticky across session restarts.
- All iOS 10 apps accessing camera, microphone, or photo library must provide `NSCameraUsageDescription`, `NSMicrophoneUsageDescription`, and/or `NSPhotoLibraryUsageDescription` in `Info.plist` or access will be denied.

---
_Source: WWDC16 Session 511 page (abstract, transcript, and resource links)._
