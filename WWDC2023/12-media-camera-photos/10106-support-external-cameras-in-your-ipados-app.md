# Support external cameras in your iPadOS app
**WWDC23 · Session 10106** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10106/)

_Platforms:_ iPadOS 17, macOS Sonoma 14 (Mac Catalyst), tvOS 17

## Overview
iPadOS 17 adds support for external USB-C cameras (UVC — USB Video Class devices) in iPad apps, including cameras built into external displays such as the Apple Studio Display and USB webcams placed on top of monitors. Apps that already use AVFoundation's AVCapture classes for built-in cameras can begin using external cameras with minimal code changes: external cameras are just `AVCaptureDevice` instances with `deviceType == .external` and `position == .unspecified`.

The session walks through evolving the AVCam sample app to discover and use external cameras, handle connect/disconnect events, adopt automatic camera selection via two new `AVCaptureDevice` class properties, apply correct video rotation with the new `AVCaptureDeviceRotationCoordinator` class, and use external microphones (including those built into displays and webcams). Best practices for format constraints, preview mirroring, multi-camera sessions, and wireless debugging close out the talk.

## Key Topics

### Discovery and Device Attributes
External cameras use three key attributes:
- **Media type:** `.video` (same as built-in cameras)
- **Device type:** `.external` **[NEW]** — replaces the deprecated `.externalUnknown` macOS type
- **Position:** `.unspecified` — because external cameras move independently of the iPad

Use `AVCaptureDeviceDiscoverySession(deviceTypes: [.external], mediaType: .video, position: .unspecified)` or query `AVCaptureDevice.default(.external, for: .video, position: .unspecified)`.

### Handling Connect/Disconnect Events
Unlike built-in cameras, external cameras can be connected and removed at any time. Apps must monitor:
- **KVO on `AVCaptureDevice.isConnected`** — observe the property on a specific device instance
- **KVO on `AVCaptureDeviceDiscoverySession.devices`** — observe the list as cameras come and go
- **`AVCaptureDeviceWasConnectedNotification` / `AVCaptureDeviceWasDisconnectedNotification`** — posted on background queues; synchronize with the AVCaptureSession queue and main thread

### Automatic Camera Selection
New in iPadOS 17, two `AVCaptureDevice` class properties automate camera selection:

- **`AVCaptureDevice.userPreferredCamera`** (read/write) — set this whenever the user picks a camera in the app; the system persists a short history of selections across launches and reboots. If the chosen camera disconnects, the system returns the next available camera from history.
- **`AVCaptureDevice.systemPreferredCamera`** (read-only, KVO-observable) — returns the best camera as determined by the system: checks user preference history first; when a new external camera is connected, it surfaces it immediately (implicit user intent). KVO this property instead of tracking individual device connect/disconnect events.

Both properties also exist on macOS (introduced in macOS Ventura for Continuity Camera); behavior described here is specific to iPadOS.

### Video Rotation with AVCaptureDeviceRotationCoordinator
`AVCaptureVideoOrientation` and all API using it are deprecated in iPadOS 17 because they assume the camera rotates with the device — invalid for external cameras. The replacement is `AVCaptureDeviceRotationCoordinator` **[NEW]**:

- **Initializer:** `init(device: AVCaptureDevice, previewLayer: CALayer?)` — pass `AVCaptureVideoPreviewLayer`, `AVSampleBufferDisplayLayer`, or the backing layer of a Metal view
- **`videoRotationAngleForHorizonLevelPreview`** (KVO, degrees) — angle to apply to the preview layer; updates arrive on the main queue; apply immediately inside the KVO handler
- **`videoRotationAngleForHorizonLevelCapture`** (KVO, degrees) — angle to embed in photos, movies, or an `AVAssetWriterInput.transform`
- Create a new coordinator each time the active capture device changes

For photo/movie outputs, set `AVCaptureConnection.videoRotationAngle`. Avoid rotating via connections for `AVCaptureVideoDataOutput` / `AVCaptureDepthDataOutput` (causes pipeline stalls and higher energy use) — instead rotate the displaying `CALayer`.

### External Microphones
iPadOS 17 improves support for external microphones connected via USB-C (including mics built into displays and webcams):
- **Telephony apps** using `AUVoiceIO` can now use external mics (previously restricted to headset mics); new echo cancellation tunings added for external mics
- **Voice Isolation mode** (Control Center) works with external mics
- The audio routing system automatically switches to the last-connected mic; `AVCaptureDevice.localizedName` changes dynamically to reflect the active route
- New `AVMediaType.microphone` device type **[NEW]** (deprecates `.builtInMicrophone`) for finding the mic device
- Use `AVAudioSession.preferredInput` to select a specific mic

### Best Practices
- Use wireless Xcode debugging when the USB-C port is occupied by an external camera
- Check `AVCaptureDevice` capabilities before use — external cameras may expose only two formats (VGA 640×480 and HD 1280×720)
- Verify `AVCaptureDevice.supportsSessionPreset(_:)` before applying presets (e.g., `.hd4K3840x2160` requires a compatible format)
- `AVCaptureVideoPreviewLayer` mirrors external cameras by default (appropriate for display-facing cameras); disable for HDMI switchers or outward-facing setups
- `AVCaptureMultiCamSession` can include external cameras for creative multi-camera capture
- Pixel format conversions happen automatically: uncompressed `yuvs`/`2vuy` → `420v`; compressed MJPEG/H.264 → `420f`

## APIs & Frameworks

- **AVFoundation** — `AVCapture*` class family
  - `AVCaptureDevice`
    - `.external` device type **[NEW]** (replaces deprecated `.externalUnknown`)
    - `class var userPreferredCamera: AVCaptureDevice?` **[NEW]** — read/write; set on user camera selection
    - `class var systemPreferredCamera: AVCaptureDevice?` **[NEW]** — read-only KVO; system-recommended best camera
    - `isConnected: Bool` — KVO-observable
    - `AVCaptureDeviceWasConnectedNotification` / `AVCaptureDeviceWasDisconnectedNotification`
  - `AVCaptureDeviceDiscoverySession`
    - `init(deviceTypes:mediaType:position:)` — existing API; now accepts `.external` device type
    - `devices: [AVCaptureDevice]` — KVO-observable; updates as cameras connect/disconnect
  - `AVCaptureDeviceRotationCoordinator` **[NEW]** — replaces `AVCaptureVideoOrientation`
    - `init(device: AVCaptureDevice, previewLayer: CALayer?)`
    - `videoRotationAngleForHorizonLevelPreview: Double` — KVO; apply to preview layer
    - `videoRotationAngleForHorizonLevelCapture: Double` — KVO; apply to capture connections or `AVAssetWriterInput.transform`
  - `AVCaptureConnection`
    - `videoRotationAngle: Double` **[NEW]** — replaces deprecated `videoOrientation`
    - `isVideoRotationAngleSupported(_ angle: Double) -> Bool` **[NEW]**
  - `AVCaptureVideoOrientation` — **deprecated** in iPadOS 17 along with all API using it
  - `AVCaptureSession` — existing central control object; unchanged API
  - `AVCaptureVideoPreviewLayer` — existing preview layer; mirrors external cameras by default
  - `AVCapturePhotoOutput` — still/Live Photo capture
  - `AVCaptureMovieFileOutput` — QuickTime movie recording
  - `AVCaptureVideoDataOutput` — raw video buffer delivery
  - `AVCaptureMultiCamSession` — multi-camera simultaneous capture; can include external cameras
  - `AVAssetWriterInput.transform` — apply capture rotation angle (converted from degrees to radians as `CGAffineTransform`) for custom movie recording
- **AVFoundation audio**
  - `AVMediaType.microphone` device type **[NEW]** — finds the active system microphone device regardless of whether it is built-in or external
  - `AVAudioSession` — `preferredInput` property for explicit mic selection
  - `AUVoiceIO` (Core Audio) — audio unit for telephony with echo cancellation; now supports external mics **[NEW tunings]**
- **AVCam sample app** — updated sample from Apple demonstrating all changes in this session (downloadable from developer.apple.com)

## Code Highlights

Finding and preferring an external camera at launch:
```swift
// Prefer external camera; fall back to built-in
if let externalCamera = AVCaptureDevice.default(.external, for: .video, position: .unspecified) {
    AVCaptureDevice.userPreferredCamera = externalCamera
}
// Observe systemPreferredCamera for connect/disconnect events
observation = AVCaptureDevice.observe(\.systemPreferredCamera, options: .new) { _, change in
    guard let newCamera = change.newValue else { return }
    DispatchQueue.main.async { self.switchCamera(to: newCamera) }
}
```

Setting up a rotation coordinator and applying preview rotation:
```swift
let coordinator = AVCaptureDeviceRotationCoordinator(device: captureDevice, previewLayer: previewLayer)
previewLayer.connection?.videoRotationAngle = coordinator.videoRotationAngleForHorizonLevelPreview

// KVO: update immediately (synchronized with system animations)
rotationObservation = coordinator.observe(\.videoRotationAngleForHorizonLevelPreview, options: .new) { _, change in
    self.previewLayer.connection?.videoRotationAngle = change.newValue ?? 0
}
```

Applying capture rotation for photos:
```swift
if let photoConnection = photoOutput.connection(with: .video) {
    photoConnection.videoRotationAngle = coordinator.videoRotationAngleForHorizonLevelCapture
}
```

## Takeaways

- External UVC cameras are first-class `AVCaptureDevice` instances with `.external` device type and `.unspecified` position; discovery uses the existing `AVCaptureDeviceDiscoverySession` API with no new framework required.
- Use `AVCaptureDevice.userPreferredCamera` (set on every user camera switch) and KVO `AVCaptureDevice.systemPreferredCamera` (auto-updates on plug/unplug) to eliminate manual connect/disconnect logic and implement automatic camera selection correctly.
- `AVCaptureDeviceRotationCoordinator` replaces the deprecated `AVCaptureVideoOrientation` API; use `videoRotationAngleForHorizonLevelPreview` for live preview and `videoRotationAngleForHorizonLevelCapture` for captured content — do not assume built-in camera rotation logic applies to external cameras.
- Check format capabilities (`supportsSessionPreset`, format count, supported pixel types) before configuring an external camera; not all presets or formats available on built-in cameras are supported by external UVC devices.

---
_Source: WWDC23 Session 10106 page (abstract, transcript, and resource links)._
