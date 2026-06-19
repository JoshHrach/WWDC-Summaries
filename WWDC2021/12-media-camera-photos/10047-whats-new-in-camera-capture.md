# What's New in Camera Capture
**WWDC21 · Session 10047** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10047/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session covers five advances in `AVFoundation`'s capture subsystem: a new `minimumFocusDistance` property for accurate macro-scanning guidance, a 10-bit HDR video format (x420) for richer dynamic range on iPhone 12 and later, three new system-level video effects (Center Stage, Portrait, Mic Modes) accessible through Control Center with no code required, a refresher on capture performance best practices, and a new IOSurface compression format that reduces memory bandwidth for hardware-accelerated pipelines.

The Video Effects section is particularly important for video conferencing apps: Center Stage (M1 iPads), Portrait (2018+ Neural Engine devices, front camera only), and Mic Modes (2018+) are injected into all apps automatically and require only opt-in or cooperative API adoption to integrate custom UI controls.

## Key Topics

### Minimum Focus Distance
`AVCaptureDevice.minimumFocusDistance` (new in iOS 15) reports the nearest distance in millimeters at which a camera can achieve sharp focus. Apps can use this to automatically apply digital zoom so users are always positioned at a scannable distance — particularly valuable for barcode scanning apps. The AVCamBarcode sample demonstrates best practices.

### 10-Bit HDR Video (x420)
Introduced on iPhone 12, the `x420` pixel format (`kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange`) identifies a true 10-bit HDR format with HLG tone curves, BT.2020 color space, and automatic per-frame Dolby Vision metadata insertion. Supported at 720p, 1080p, 4K, and 1920×1440. Locate the format by iterating `AVCaptureDevice.formats` and matching the pixel format type.

### Video Effects in Control Center
Three system effects are now available per-app with no code changes. Center Stage (M1 iPad Pro) uses the 12 MP Ultra Wide camera to pan and track the subject. Portrait provides Neural Engine-powered shallow depth of field. Mic Modes offer Standard, Wide Spectrum, or Voice Isolation audio processing. All are toggled by the user in Control Center; apps can observe state and optionally present cooperative UI. Portrait and Mic Modes require app opt-in on iOS (except VoIP apps); macOS apps are opted in automatically. `AVCaptureDevice.showSystemUserInterface(_:)` deep-links to the relevant Control Center module.

### Capture Performance Best Practices
`AVCaptureVideoDataOutput.alwaysDiscardsLateVideoFrames` (default `true`) enforces a one-buffer queue. The `captureOutput(_:didDrop:from:)` delegate provides `CMSampleBuffer` attachments with `.droppedFrameReason` values: `FrameWasLate`, `OutOfBuffers`, `Discontinuity`. Mitigation strategies include lowering `AVCaptureDevice.activeMinVideoFrameDuration` or reducing workload. `AVCaptureDevice.systemPressureState` reports thermal/power pressure levels (nominal → fair → serious → critical → shutdown).

### IOSurface Compression
New in iOS 15 on iPhone 12+, Fall 2020 iPad Air, and M1 iPad Pro: a lossless in-memory compression format that reduces memory bandwidth for hardware-only pipelines. Compressed pixel format types are available for 420v, 420f, x420, and BGRA. Request them via `AVCaptureVideoDataOutput.videoSettings`. Rules: do not write to disk, do not assume a fixed layout, do not access via CPU.

## APIs & Frameworks

**AVFoundation / AVCapture**

- `AVCaptureDevice.minimumFocusDistance: Int` **[NEW]** — minimum focus distance in mm; -1 if fixed-focus
- `kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange` (`x420`) — 10-bit HDR format; available via `AVCaptureDevice.formats`
- `AVCaptureDevice.isCenterStageEnabled: Bool` **[NEW]** (class property) — read/write with KVO support
- `AVCaptureDevice.centerStageControlMode: AVCaptureDevice.CenterStageControlMode` **[NEW]** — `.user`, `.app`, `.cooperative`
- `AVCaptureDevice.Format.isCenterStageSupported: Bool` **[NEW]**
- `AVCaptureDevice.isCenterStageActive: Bool` **[NEW]** — per-instance KVO property
- `AVCaptureDevice.isPortraitEffectActive: Bool` **[NEW]** — KVO observable
- `AVCaptureDevice.Format.isPortraitEffectSupported: Bool` **[NEW]**
- `NSCameraPortraitEffectEnabled` (Info.plist key) **[NEW]** — opt-in for non-VoIP iOS apps
- `AVCaptureDevice.preferredMicrophoneMode: AVCaptureDevice.MicrophoneMode` **[NEW]** — user's selected mode
- `AVCaptureDevice.activeMicrophoneMode: AVCaptureDevice.MicrophoneMode` **[NEW]** — active mode given current route
- `AVCaptureDevice.MicrophoneMode` **[NEW]** — `.standard`, `.wideSpectrum`, `.voiceIsolation`
- `AVCaptureDevice.showSystemUserInterface(_:)` **[NEW]** — `.videoEffects` or `.microphoneModes`
- `AVCaptureVideoDataOutput.alwaysDiscardsLateVideoFrames: Bool` — existing best-practice property
- `CMSampleBuffer` attachment `.droppedFrameReason` — `FrameWasLate`, `OutOfBuffers`, `Discontinuity`
- `AVCaptureDevice.systemPressureState: AVCaptureDevice.SystemPressureState` — existing
- `AVCaptureDevice.activeMinVideoFrameDuration: CMTime` — frame rate throttle
- IOSurface-compressed pixel format types (iOS 15) **[NEW]**: compressed variants of 420v, 420f, x420, BGRA

## Code Highlights

Calculating minimum zoom for barcode scanning:
```swift
let deviceMinimumFocusDistance = Float(videoDeviceInput.device.minimumFocusDistance)
if minimumSubjectDistanceForCode < deviceMinimumFocusDistance {
    let zoomFactor = deviceMinimumFocusDistance / minimumSubjectDistanceForCode
    try videoDeviceInput.device.lockForConfiguration()
    videoDeviceInput.device.videoZoomFactor = CGFloat(zoomFactor)
    videoDeviceInput.device.unlockForConfiguration()
}
```

Finding 10-bit HDR format:
```swift
func firstTenBitFormatOfDevice(device: AVCaptureDevice) -> AVCaptureDevice.Format? {
    for format in device.formats {
        let pixelFormat = CMFormatDescriptionGetMediaSubType(format.formatDescription)
        if pixelFormat == kCVPixelFormatType_420YpCbCr10BiPlanarVideoRange {
            return format
        }
    }
    return nil
}
```

## Takeaways
- `AVCaptureDevice.minimumFocusDistance` enables barcode/QR scanning apps to automatically apply the right zoom factor, eliminating blurry near-focus failures.
- 10-bit HDR video (`x420`) is available on iPhone 12+; select it by matching the pixel format type when iterating device formats — Dolby Vision metadata is inserted automatically.
- Center Stage, Portrait, and Mic Modes are system-level effects that appear in all apps via Control Center with zero code; use cooperative mode and KVO to integrate them into your app's own UI.
- IOSurface compression (iOS 15, iPhone 12+) transparently reduces memory bandwidth for hardware pipelines; opt into compressed pixel format types in `AVCaptureVideoDataOutput.videoSettings` for maximum benefit.

---
_Source: WWDC21 Session 10047 page (abstract, chapter summaries, code samples, and resource links)._
