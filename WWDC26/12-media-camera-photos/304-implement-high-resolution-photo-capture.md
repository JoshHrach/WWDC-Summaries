# Implement High Resolution Photo Capture
**WWDC26 · Session 304** · [Watch](https://developer.apple.com/videos/play/wwdc2026/304/)

_Platforms:_ iOS, iPadOS

## Overview
iPhone cameras now offer 12MP, 24MP, and 48MP capture across the Main, Tele, and Ultra Wide lenses. This session walks through the AVFoundation APIs required to unlock those higher resolutions, covering the four capture types available (fully processed, exposure-bracketed, Bayer RAW, and Apple ProRAW), and how to configure `AVCaptureSession` and `AVCapturePhotoOutput` to target a specific maximum dimension.

Responsive capture is the core performance theme. The session introduces deferred photo processing, overlapping captures, and fast capture prioritization as the three tools that keep shot-to-shot delay low even when capturing 48MP ProRAW frames.

Preallocating pipeline resources with `setPreparedPhotoSettingsArray(_:completionHandler:)` before the first capture is emphasized as the single most impactful step for eliminating first-shot latency.

## Key Topics

### High-Resolution Photos
The photonic engine balances light gathering and fine detail across resolutions. 24MP and 48MP modes are available on the Main camera; 48MP is also available on Ultra Wide on some models. Tele cameras support up to 12MP on most iPhone variants.

### Types of Captures
- **Fully processed** — system applies the photonic engine, Smart HDR, and computational photography; the easiest path for most apps.
- **Exposure brackets** — app receives multiple frames at different EV values for custom HDR pipelines.
- **Bayer RAW** — sensor-native data, minimal processing, maximum flexibility.
- **Apple ProRAW** — computational photography metadata embedded in a DNG for later re-processing.

### Configure a Capture Session
Set `sessionPreset = .photo`, configure `maxPhotoQualityPrioritization`, then set `maxPhotoDimensions` on the output to the largest supported dimension. Commit the configuration before calling `startRunning()`.

### Responsive Capture Best Practices
Use `setPreparedPhotoSettingsArray` to warm the pipeline before the first shot. Enable `isResponsiveCaptureEnabled` to allow overlapping captures. Use fast capture prioritization when shutter speed matters more than quality. Deferred photo processing lets the system finish expensive work after the next shot is already queued.

## APIs & Frameworks

### AVFoundation
- `AVCaptureSession` — top-level session object
  - `sessionPreset` — set to `.photo` for highest quality
  - `beginConfiguration()` / `commitConfiguration()`
  - `startRunning()`
- `AVCapturePhotoOutput` — manages photo pipeline
  - `maxPhotoQualityPrioritization` — `.quality` or `.balanced`
  - `maxPhotoDimensions` — `CMVideoDimensions` for the output; set to largest supported
  - `isResponsiveCaptureEnabled` — enable overlapping/fast captures
  - `isResponsiveCaptureSupported` — guard before enabling
  - `setPreparedPhotoSettingsArray(_:completionHandler:)` — preallocate pipeline resources
  - `capturePhoto(with:delegate:)` — trigger a capture
- `AVCapturePhotoSettings`
  - `maxPhotoDimensions` — per-capture dimension override
  - `photoQualityPrioritization` — `.quality`, `.balanced`, `.speed`
- `AVCaptureDevice`
  - `activeFormat.supportedMaxPhotoDimensions` — array of `CMVideoDimensions` available on the active format
- `AVCapturePhotoCaptureDelegate` — receive captured photo data
- `AVCaptureVideoPreviewLayer` — live preview rendering
- Capture types: fully processed, exposure brackets, Bayer RAW (`rawPhotoPixelFormatType`), Apple ProRAW (`isAppleProRAWEnabled`)

### Sample Code Reference
- [AVCam: Building a camera app](https://developer.apple.com/documentation/AVFoundation/avcam-building-a-camera-app)
- [Capturing photos in RAW and Apple ProRAW formats](https://developer.apple.com/documentation/AVFoundation/capturing-photos-in-raw-and-apple-proraw-formats)

## Code Highlights

Configure output and pick largest dimension:
```swift
let supportedMaxPhotoDimensions = device?.activeFormat.supportedMaxPhotoDimensions ?? []
if let largestDimension = supportedMaxPhotoDimensions.max(by: { lhs, rhs in
    Int(lhs.width) * Int(lhs.height) < Int(rhs.width) * Int(rhs.height)
}) {
    photoOutput?.maxPhotoDimensions = largestDimension
}
```

Preallocate pipeline then capture with fresh settings:
```swift
let prepareSettings = AVCapturePhotoSettings()
prepareSettings.maxPhotoDimensions = photoOutput.maxPhotoDimensions
prepareSettings.photoQualityPrioritization = .quality
photoOutput.setPreparedPhotoSettingsArray([prepareSettings]) { prepared, error in … }

// Later — new settings object required
let captureSettings = AVCapturePhotoSettings()
captureSettings.maxPhotoDimensions = photoOutput.maxPhotoDimensions
captureSettings.photoQualityPrioritization = .quality
photoOutput.capturePhoto(with: captureSettings, delegate: self)
```

## Takeaways
- Always call `setPreparedPhotoSettingsArray` before the first capture to eliminate pipeline warm-up latency.
- Use `activeFormat.supportedMaxPhotoDimensions` to discover 24MP/48MP availability at runtime rather than hardcoding dimensions.
- `isResponsiveCaptureEnabled` unlocks overlapping captures — essential for burst-like UX at high resolutions.
- A new `AVCapturePhotoSettings` instance must be created for each capture; reusing a prepared-settings object will fail.

---
_Source: WWDC26 Session 304 page (abstract, chapter summaries, code samples, and resource links)._
