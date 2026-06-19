# Capture High-Quality Photos Using Video Formats
**WWDC21 · Session 10247** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10247/)

_Platforms:_ iOS 15, iPadOS 15

## Overview
iOS 15 delivers a major leap in photo quality when capturing with video formats in AVFoundation, using improved algorithms that apply noise reduction and multi-frame fusion techniques previously reserved for photo formats — now without impacting video recording continuity or live preview. This session explains the photo quality vs. speed trade-off, introduces `AVCapturePhotoOutput.QualityPrioritization` (originally added in iOS 13), and covers the new `isHighPhotoQualitySupported` format property that identifies video formats eligible for the upgraded pipeline.

Photo formats (session preset `.photo`, `isHighestPhotoQualitySupported == true`) give access to Live Photo, ProRAW, and the highest resolutions at up to 30 fps. Video formats give access to high frame rates (60 fps), flexible resolutions, and lower overhead for real-time processing — and as of iOS 15, also deliver significantly better photo quality at `.balanced` and `.quality` prioritization.

## Key Topics

### Photo vs. Video Formats
- **Photo formats**: `isHighestPhotoQualitySupported == true`; best quality, max 30 fps; exclusive features: Live Photo, ProRAW, portrait effects matte.
- **Video formats**: `isHighestPhotoQualitySupported == false`; higher frame rates (up to 60 fps), lower overhead, suited for apps doing real-time computation.
- `AVCaptureVideoDataOutput` sample buffer resolution depends on format type: preview-only resolution for photo formats, full resolution for video formats.

### Quality Prioritization (iOS 13+)
`AVCapturePhotoOutput.QualityPrioritization` specifies the quality-vs-speed trade-off:
- `.speed` — WYSIWYG, minimal processing, fastest delivery; no frame drops, no preview interruption.
- `.balanced` — fast multi-frame fusion algorithms, noticeably better quality with minor latency; on iOS 15 video formats: greatly improved quality with no frame drops.
- `.quality` — heaviest algorithms (Deep Fusion, etc.), best quality; may cause frame drops on video formats depending on device.

`maxPhotoQualityPrioritization` on `AVCapturePhotoOutput` must be set before session start; per-capture `photoQualityPrioritization` on `AVCapturePhotoSettings` cannot exceed this ceiling.

`photoProcessingTimeRange` on `AVCaptureResolvedPhotoSettings` (received in delegate callbacks before delivery) indicates expected processing duration, useful for deciding whether to show a spinner.

### New iOS 15: High Photo Quality for Video Formats
Supported video formats now carry `isHighPhotoQualitySupported == true`. These formats receive improved photo quality at `.balanced` and `.quality` prioritization on iPhone XS and later. Supported resolutions: 1280×720 (30/60 fps), 1920×1080 (30/60 fps), 1920×1440 (30 fps), 3840×2160 / 4K (30 fps).

Apps compiled before iOS 15 that used `.quality` with video formats are automatically downgraded to `.balanced` at runtime; to opt into `.quality` behavior, recompile against the iOS 15 SDK.

### Caveats
- Feature works with `AVCaptureSession` only, not `AVCaptureMultiCamSession`.
- `AVCaptureStillImageOutput` (deprecated) does not support this feature.
- `.balanced`/`.quality` may produce photos that look different from simultaneously recorded video frames due to multi-exposure fusion; use `.speed` if exact match is required.

## APIs & Frameworks

**AVFoundation** (`import AVFoundation`)

- `AVCaptureSession` — session graph root
- `AVCaptureDevice` — camera device
- `AVCaptureDeviceInput` — wraps device for session input
- `AVCapturePhotoOutput` — still photo capture output
  - `maxPhotoQualityPrioritization` — `.speed` / `.balanced` / `.quality`; set before session start
- `AVCapturePhotoOutput.QualityPrioritization` **[iOS 13]** — enum: `.speed`, `.balanced`, `.quality`
- `AVCapturePhotoSettings` — per-capture configuration
  - `photoQualityPrioritization` — per-capture prioritization (≤ `maxPhotoQualityPrioritization`)
- `AVCaptureResolvedPhotoSettings` — resolved settings delivered in delegate
  - `photoProcessingTimeRange` — `CMTimeRange` indicating expected delivery window
- `AVCapturePhoto` — delivered captured photo object
- `AVCapturePhotoCaptureDelegate` — delegate protocol for capture lifecycle
- `AVCapturePhotoOutput.capturePhoto(with:delegate:)` — trigger capture
- `AVCaptureDevice.Format` — camera format descriptor
  - `isHighestPhotoQualitySupported` — `true` for photo formats
  - `isHighPhotoQualitySupported` **[NEW iOS 15]** — `true` for video formats supporting improved photo quality algorithms
- `AVCaptureSession.sessionPreset` — `.photo` selects photo format preset
- `AVCaptureStillImageOutput` — deprecated; does not support quality prioritization improvements

## Code Highlights

Setting up quality prioritization on the photo output:
```swift
// Set once before starting the session
photoOutput.maxPhotoQualityPrioritization = .quality

// Per-capture: must be ≤ maxPhotoQualityPrioritization
let settings = AVCapturePhotoSettings()
settings.photoQualityPrioritization = .balanced
photoOutput.capturePhoto(with: settings, delegate: self)
```

Finding a video format with high photo quality support:
```swift
guard let format = device.formats.first(where: { $0.isHighPhotoQualitySupported }) else { return }
try device.lockForConfiguration()
device.activeFormat = format
device.unlockForConfiguration()
```

Using `photoProcessingTimeRange` to decide UI treatment:
```swift
func photoOutput(_ output: AVCapturePhotoOutput,
                 willBeginCaptureFor resolvedSettings: AVCaptureResolvedPhotoSettings) {
    let range = resolvedSettings.photoProcessingTimeRange
    if range.duration > CMTime(seconds: 0.5, preferredTimescale: 1000) {
        showSpinner()
    }
}
```

## Takeaways
- `AVCapturePhotoOutput.QualityPrioritization` (`.speed`, `.balanced`, `.quality`) is the unified API for controlling photo quality across all format types.
- iOS 15 brings significantly improved photo quality to popular video formats (720p through 4K) at `.balanced` prioritization with no impact on concurrent video recording or preview.
- The new `isHighPhotoQualitySupported` property identifies video formats that participate in the improved pipeline.
- Apps using `.balanced` with `AVCapturePhotoOutput` automatically benefit on iOS 15 with no code changes required.

---
_Source: WWDC21 Session 10247 page (abstract, chapter summaries, code samples, and resource links)._
