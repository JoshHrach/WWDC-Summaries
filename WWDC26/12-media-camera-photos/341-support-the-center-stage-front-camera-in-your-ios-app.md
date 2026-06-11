# Support the Center Stage Front Camera in Your iOS App
**WWDC26 · Session 341** · [Watch](https://developer.apple.com/videos/play/wwdc2026/341/)

_Platforms:_ iOS (iPhone 17, iPhone 17 Pro, iPhone Air)

## Overview
iPhone 17, iPhone 17 Pro, and iPhone Air introduce a Center Stage front camera with a square image sensor and a 95-degree field of view. The square sensor means the camera captures usable pixels in all orientations without rotating the device, and the wide FOV gives apps room to zoom, crop, and reframe automatically without quality loss.

This session introduces the AVFoundation APIs that unlock three new capabilities: dynamic aspect ratio switching (without interrupting the capture session), a smart framing monitor that recommends zoom and crop based on face and gaze detection, and Center Stage for video calls (cooperative and app-controlled modes with low-latency stabilization).

Sensor orientation compensation — the camera's native portrait orientation — is handled automatically by `AVCapturePhotoOutput` but requires attention when using `AVCaptureMovieFileOutput` or `AVAssetWriter` for video.

## Key Topics

### Center Stage Front Camera Hardware
The square sensor captures a 1:1 image that the system crops to the requested aspect ratio. The 95-degree FOV accommodates both portrait selfies and landscape group shots from the same physical position.

### Center Stage for Photos — Auto Zoom and Auto Rotate
Without any API changes, the system applies face and gaze detection to automatically zoom and crop still photos. Apps that want fine control use the dynamic aspect ratio and smart framing APIs below.

### Capture Session Setup
Use `AVCaptureDevice.DiscoverySession` with `deviceTypes: [.builtInUltraWideCamera]` and `position: .front` to find the Center Stage front camera. Wire it up to `AVCapturePhotoOutput` and `AVCaptureVideoPreviewLayer` as normal.

### Dynamic Aspect Ratio
Query `format.supportedDynamicAspectRatios` to find supported ratios (e.g., `.ratio4x3`, `.ratio16x9`, `.ratio1x1`). Call `camera.setDynamicAspectRatio(_:)` — an async throwing method — to switch ratios mid-session. The call returns the `CMTime` timestamp at which the change takes effect, enabling precise UI synchronization.

### Smart Framing Monitor
`AVCaptureSmartFramingMonitor` (accessed via `camera.smartFramingMonitor`) periodically publishes `recommendedFraming` recommendations containing an aspect ratio and zoom factor derived from face and gaze detection. Use KVO on `recommendedFraming` to receive updates, then apply them with `setDynamicAspectRatio` and `camera.videoZoomFactor`. Call `startMonitoring()` and `stopMonitoring()` to control its lifetime.

### Sensor Orientation Compensation
The sensor's native orientation is portrait. `AVCapturePhotoOutput` compensates automatically. For video, apps using `AVCaptureMovieFileOutput` or `AVAssetWriter` must explicitly account for the sensor orientation in the video track transform.

### Center Stage for Video Recordings
Apply the same dynamic aspect ratio API to video via `AVCaptureMovieFileOutput`. Cinematic stabilization modes are available for smooth footage.

### Center Stage for Video Calls
Set `AVCaptureDevice.centerStageControlMode = .cooperative` (shares control with the user) or `.app` (app has full control). Set `AVCaptureDevice.isCenterStageEnabled = true`. Enable low-latency stabilization for significantly smoother real-time video conferencing output.

## APIs & Frameworks

### AVFoundation
- `AVCaptureDevice.DiscoverySession` — used with `.builtInUltraWideCamera` + `.front` to discover Center Stage camera
- `AVCaptureDevice`
  - `formats` — iterable collection of `AVCaptureDevice.Format`
  - `activeFormat` — set after finding the appropriate format
  - `lockForConfiguration()` / `unlockForConfiguration()`
  - `setDynamicAspectRatio(_:)` — **[NEW]** async; returns effective `CMTime`
  - `videoZoomFactor` — apply smart framing zoom recommendations
  - `smartFramingMonitor` — **[NEW]** optional `AVCaptureSmartFramingMonitor`
  - `isCenterStageEnabled` — static; enable Center Stage for video calls
  - `centerStageControlMode` — **[NEW]** `.cooperative` or `.app`
- `AVCaptureDevice.Format`
  - `supportedDynamicAspectRatios` — **[NEW]** array of `AVCaptureDevice.DynamicAspectRatio`
  - `isSmartFramingSupported` — **[NEW]** Bool
  - `isCenterStageSupported` — Bool; existing property
- `AVCaptureDevice.DynamicAspectRatio` — **[NEW]** enum: `.ratio4x3`, `.ratio16x9`, `.ratio1x1`, etc.
- `AVCaptureSmartFramingMonitor` — **[NEW]** class
  - `supportedFramings` — all framings the monitor can recommend
  - `enabledFramings` — subset the app activates
  - `recommendedFraming` — KVO-observable optional `AVCaptureSmartFraming`
  - `startMonitoring()` / `stopMonitoring()`
- `AVCaptureSmartFraming` — **[NEW]** value type with `aspectRatio` and `zoomFactor`
- `AVCapturePhotoOutput` — automatic sensor orientation compensation (no extra work required)
- `AVCaptureMovieFileOutput` — video recording; orientation compensation and cinematic stabilization
- `AVAssetWriter` — alternative video recording path; manual orientation compensation required

### Resources
- [Supporting Center Stage front camera in your iOS app](https://developer.apple.com/documentation/AVFoundation/supporting-center-stage-front-camera-in-your-ios-app)
- [AVCam: Building a camera app](https://developer.apple.com/documentation/AVFoundation/avcam-building-a-camera-app)
- [Capture setup](https://developer.apple.com/documentation/AVFoundation/capture-setup)

## Code Highlights

Select a format that supports 4:3 and apply it:
```swift
for format in camera.formats {
    if format.supportedDynamicAspectRatios.contains(.ratio4x3) {
        try! camera.lockForConfiguration()
        camera.activeFormat = format
        camera.unlockForConfiguration()
        break
    }
}
let timestamp = try! await camera.setDynamicAspectRatio(.ratio4x3)
```

Apply smart framing recommendations via KVO:
```swift
observation = monitor.observe(\.recommendedFraming, options: [.new]) { monitor, _ in
    if let framing = monitor.recommendedFraming {
        Task {
            try! camera.lockForConfiguration()
            try! await camera.setDynamicAspectRatio(framing.aspectRatio)
            camera.videoZoomFactor = CGFloat(framing.zoomFactor)
            camera.unlockForConfiguration()
        }
    }
}
try! monitor.startMonitoring()
```

Enable Center Stage for video calls:
```swift
AVCaptureDevice.centerStageControlMode = .cooperative
AVCaptureDevice.isCenterStageEnabled = true
```

## Takeaways
- The Center Stage front camera is available on iPhone 17, iPhone Air, and iPhone 17 Pro; use `AVCaptureDevice.DiscoverySession` with `.builtInUltraWideCamera` + `.front` to find it.
- `setDynamicAspectRatio(_:)` switches aspect ratios without rebuilding the capture session — use the returned timestamp for precise UI transitions.
- `AVCaptureSmartFramingMonitor` automates framing decisions; apps only need to apply the recommended aspect ratio and zoom factor.
- For video calls, set `centerStageControlMode` before enabling `isCenterStageEnabled`; `.cooperative` respects user overrides while `.app` gives full programmatic control.

---
_Source: WWDC26 Session 341 page (abstract, chapter summaries, code samples, and resource links)._
