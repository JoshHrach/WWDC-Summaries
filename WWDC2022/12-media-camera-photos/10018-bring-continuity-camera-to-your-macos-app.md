# Bring Continuity Camera to Your macOS App
**WWDC22 · Session 10018** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10018/)

_Platforms:_ macOS Ventura 13, iOS 16

## Overview
Continuity Camera turns iPhone into a wireless or wired external camera and microphone for any Mac app running macOS 13 paired with an iPhone running iOS 16 (same Apple ID, two-factor authentication enabled). The iPhone appears as a standard `AVCaptureDevice` on the Mac, requiring no special integration for basic camera use — apps already using `AVFoundation` camera APIs get Continuity Camera automatically.

The session covers three areas: the out-of-the-box experience (new system video effects, Desk View), new APIs for automatic camera selection (`userPreferredCamera`, `systemPreferredCamera`), and macOS-only AVFoundation additions that enable high-resolution photo capture, flash, metadata output, and Desk View camera integration.

System video effects supported by Continuity Camera include Center Stage, Portrait (now available on Intel Macs via iPhone), and new Studio Light (iPhone 12 or newer).

## Key Topics

### Automatic Camera Selection
- `AVCaptureDevice.userPreferredCamera` — read/write; set whenever a user selects a camera; persisted across app launches; KVO-observable; falls back to next available camera when preferred device disconnects
- `AVCaptureDevice.systemPreferredCamera` — read-only; incorporates user preference plus system signals (Continuity Camera proximity/position, device suspension from lid close); KVO-observable
- Continuity Camera triggers automatic selection when iPhone is on a stationary landscape stand, screen off, and connected (USB or proximity with BT+Wi-Fi)
- Recommended: support both "Auto" mode (KVO on `systemPreferredCamera`) and manual mode (set `userPreferredCamera` on user pick)

### High-Resolution Photo Capture
- `AVCapturePhotoOutput.highResolutionCaptureEnabled = true` (before starting session) — enables up to 12 MP captures
- `AVCapturePhotoSettings.highResolutionPhotoEnabled = true` — set per capture
- `AVCapturePhotoOutput.maxPhotoQualityPrioritization` — set max quality level on output
- `AVCapturePhotoSettings.photoQualityPrioritization` — set per capture (speed vs. quality tradeoff)
- `AVCapturePhotoSettings.flashMode` — `.on`, `.off`, `.auto`

### Metadata Output (new on macOS)
- `AVCaptureMetadataOutput` now available on macOS 13 **[NEW]**
- Supports face metadata objects (`AVMetadataObject.ObjectType.face`) and human body metadata objects
- Setup: add output to session, set `metadataObjectTypes`, set delegate
- Callbacks via `AVCaptureMetadataOutputObjectsDelegate`

### Desk View Camera API
- `AVCaptureDevice.companionDeskViewCamera: AVCaptureDevice?` — access Desk View camera from main camera device **[NEW]**
- Discovery: `AVCaptureDevice.DiscoverySession` with `AVCaptureDevice.DeviceType.deskViewCamera` **[NEW]**
- Supported outputs: `AVCaptureVideoDataOutput`, `AVCaptureMovieFileOutput`, `AVCaptureVideoPreviewLayer`
- Format: 1920×1440, 420v pixel format, up to 30 fps

### Continuity Camera Supported Formats
- 640×480, 1280×720, 1920×1080 (16:9) — up to 30 fps or 60 fps
- 1920×1440 (4:3) — up to 30 fps or 60 fps

## APIs & Frameworks

**AVFoundation — Camera Selection** **[NEW]**
- `AVCaptureDevice.userPreferredCamera: AVCaptureDevice?` — class property, read/write, KVO **[NEW]**
- `AVCaptureDevice.systemPreferredCamera: AVCaptureDevice?` — class property, read-only, KVO **[NEW]**

**AVFoundation — Photo Capture (macOS additions)**
- `AVCapturePhotoOutput.highResolutionCaptureEnabled: Bool` — enables high-resolution capture **[updated for macOS]**
- `AVCapturePhotoSettings.highResolutionPhotoEnabled: Bool` — per-capture high-resolution
- `AVCapturePhotoOutput.maxPhotoQualityPrioritization: AVCapturePhotoOutput.QualityPrioritization`
- `AVCapturePhotoSettings.photoQualityPrioritization: AVCapturePhotoOutput.QualityPrioritization`
- `AVCapturePhotoSettings.flashMode: AVCaptureDevice.FlashMode` — `.on`, `.off`, `.auto`

**AVFoundation — Metadata Output (new on macOS)** **[NEW]**
- `AVCaptureMetadataOutput` — now available on macOS 13 **[NEW]**
- `AVCaptureMetadataOutput.metadataObjectTypes: [AVMetadataObject.ObjectType]`
- `AVCaptureMetadataOutput.availableMetadataObjectTypes: [AVMetadataObject.ObjectType]`
- `AVCaptureMetadataOutputObjectsDelegate` protocol
  - `metadataOutput(_:didOutput:from:)`
- `AVMetadataObject.ObjectType.face`
- `AVMetadataObject.ObjectType.humanBody`

**AVFoundation — Desk View** **[NEW]**
- `AVCaptureDevice.DeviceType.deskViewCamera` **[NEW]**
- `AVCaptureDevice.companionDeskViewCamera: AVCaptureDevice?` **[NEW]**
- `AVCaptureDevice.DiscoverySession(deviceTypes:mediaType:position:)` — enumerate Desk View cameras

**System Video Effects (no API required)**
- Center Stage (AVFoundation-exposed on iOS 14.5+, macOS 12.3+)
- Portrait (now on Intel Macs via Continuity Camera, previously Apple silicon only)
- Studio Light **[NEW]** — macOS 13, requires iPhone 12 or newer

## Code Highlights

Automatic camera switching with KVO:
```swift
// Auto mode: observe systemPreferredCamera
AVCaptureDevice.addObserver(self, forKeyPath: "systemPreferredCamera",
                             options: .new, context: nil)

override func observeValue(forKeyPath keyPath: String?, ...) {
    if keyPath == "systemPreferredCamera",
       let newCamera = AVCaptureDevice.systemPreferredCamera {
        updateSessionInputDevice(newCamera)
    }
}

// When user manually picks a camera:
AVCaptureDevice.userPreferredCamera = userSelectedDevice
```

Enabling high-resolution photo capture:
```swift
photoOutput.highResolutionCaptureEnabled = true
let settings = AVCapturePhotoSettings()
settings.highResolutionPhotoEnabled = true
settings.photoQualityPrioritization = .quality
settings.flashMode = .auto
photoOutput.capturePhoto(with: settings, delegate: self)
```

Setting up metadata output for faces on macOS:
```swift
let metadataOutput = AVCaptureMetadataOutput()
session.addOutput(metadataOutput)
if metadataOutput.availableMetadataObjectTypes.contains(.face) {
    metadataOutput.metadataObjectTypes = [.face]
}
metadataOutput.setMetadataObjectsDelegate(self, queue: .main)
```

Accessing the Desk View camera:
```swift
// From main camera device
if let deskView = mainCamera.companionDeskViewCamera {
    // Add to session
}
// Or via discovery
let session = AVCaptureDevice.DiscoverySession(
    deviceTypes: [.deskViewCamera], mediaType: .video, position: .unspecified)
```

## Takeaways
- Existing macOS AVFoundation apps get Continuity Camera automatically — no code changes needed for basic functionality.
- Adopt `systemPreferredCamera` KVO to enable seamless automatic switching when an iPhone is placed in Continuity Camera position; always set `userPreferredCamera` when the user manually picks a device.
- High-resolution photo capture (up to 12 MP) from iPhone via Continuity Camera is now possible on macOS using existing `AVCapturePhotoOutput` APIs.
- `AVCaptureMetadataOutput` is new on macOS 13, enabling face and body metadata streaming from Continuity Camera.

---
_Source: WWDC22 Session 10018 page (abstract, chapter summaries, code samples, and resource links)._
