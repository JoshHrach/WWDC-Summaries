# Advances in iOS Photography
**WWDC16 · Session 501** · [Watch](https://developer.apple.com/videos/play/wwdc2016/501/)

_Platforms:_ iOS 10

## Overview
This session introduces `AVCapturePhotoOutput`, a brand-new capture output in iOS 10 that replaces the deprecated `AVCaptureStillImageOutput`. The new API addresses the old class's design limitations with a functional programming model, immutable resolved settings, and a clean delegate callback chain that tracks each photo request from submission to completion.

The session covers four major feature areas: capturing Live Photos with the same quality and frictionless UX as Apple's Camera app; capturing RAW images and storing them as DNG files (a first on the platform); obtaining preview/thumbnail images alongside full-resolution photos for a responsive UI; and capturing wide-color photos using the Display P3 color space on iPad Pro 9.7.

A companion Chalk Talk (Session 511) covers scene monitoring, resource preparation/reclamation, and camera privacy policy changes in iOS 10.

## Key Topics

### AVCapturePhotoOutput (New)
- Single `capturePhoto(_:delegate:)` verb replaces multiple StillImageOutput methods.
- Per-capture settings encapsulated in `AVCapturePhotoSettings`; each instance has a unique ID and may only be used once.
- Resolved settings delivered early via `AVCaptureResolvedPhotoSettings` so app can pre-allocate before the photo arrives.
- Ordered, documented delegate callback chain (willBeginCapture → willCapture → didCapture → didFinishProcessing → didFinishCapture).
- Features that affect the render pipeline (high-resolution capture, Live Photo) must be opted into before `startRunning()`.

### Live Photos
- Captures a 12 MP JPEG and a stabilized ~3-second QuickTime movie with audio; still shares a UUID in EXIF Maker Note and top-level movie metadata.
- New in iOS 10: all Live Photo movies are stabilized; music playback is uninterrupted during capture.
- `isLivePhotoCaptureEnabled` must be set before session start; requires microphone `AVCaptureDeviceInput`.
- `isLivePhotoCaptureSuspended` allows apps to bracket audio events cleanly.
- Editing Live Photos without losing the movie track is now supported (iOS 10 Photos framework).

### RAW Photo Capture
- First RAW support on iOS; four Bayer-pattern pixel format constants added to `CVPixelBuffer.h`.
- RAW-only or RAW + processed (JPEG/BGRA/420) simultaneous capture supported.
- RAW brackets supported; RAW + SIS not supported.
- Output stored via `AVCapturePhotoOutput.dngPhotoDataRepresentation(_:previewPhotoSampleBuffer:)` as compressed DNG.

### Preview / Thumbnail Images
- Uncompressed preview (420fv or BGRA) delivered alongside full-resolution photo without extra decompress/downscale steps.
- Caller specifies desired bounding box dimensions; PhotoOutput preserves aspect ratio.
- Preview can be embedded as a thumbnail in DNG files.

### Wide Color (Display P3)
- iPad Pro 9.7 supports Display P3 (same primaries as DCI P3; gamma and white point matching sRGB at 6500 K).
- `AVCaptureDevice.Format.supportedColorSpaces` **[NEW]** — array containing `.sRGB` (0) or `.P3_D65` (1).
- `AVCaptureDevice.activeColorSpace` **[NEW]** — settable to `.P3_D65` on supported formats (420f full-range only).
- `AVCaptureSession.automaticallyConfiguresCaptureDeviceForWideColor` **[NEW]** — automatically selects P3 when a PhotoOutput is present and no ambiguous video outputs exist.
- Wide color supported in JPEG, BGRA, 420f, and Live Photos; 420v is converted to sRGB.
- RAW capture is inherently wide color (sensor primaries stored verbatim; rendering choice deferred to post).

## APIs & Frameworks

- **AVFoundation**
- `AVCapturePhotoOutput` **[NEW]** — replaces `AVCaptureStillImageOutput` (deprecated in iOS 10)
- `AVCapturePhotoSettings` **[NEW]** — per-capture settings object with unique ID
- `AVCapturePhotoBracketSettings` **[NEW]** — subclass for bracketed captures
- `AVCaptureResolvedPhotoSettings` **[NEW]** — immutable resolved settings passed in every delegate callback
- `AVCapturePhotoCaptureDelegate` **[NEW]** — ordered callback protocol
  - `photoOutput(_:willBeginCaptureFor:)` — first callback; resolved settings available
  - `photoOutput(_:willCapturePhotoFor:)` — shutter moment
  - `photoOutput(_:didCapturePhotoFor:)` — sensor readout complete
  - `photoOutput(_:didFinishProcessingPhoto:previewPhoto:resolvedSettings:bracketSettings:error:)` — processed image delivered
  - `photoOutput(_:didFinishProcessingRawPhoto:previewPhoto:resolvedSettings:bracketSettings:error:)` **[NEW]** — RAW image delivered
  - `photoOutput(_:didFinishRecordingLivePhotoMovieForEventualFileAt:resolvedSettings:)` **[NEW]** — Live Photo samples collected
  - `photoOutput(_:didFinishProcessingLivePhotoToMovieFileAt:duration:photoDisplayTime:resolvedSettings:error:)` **[NEW]** — Live Photo movie written
  - `photoOutput(_:didFinishCaptureFor:error:)` — final cleanup callback
- `AVCapturePhotoOutput.capturePhoto(with:delegate:)` **[NEW]**
- `AVCapturePhotoOutput.isLivePhotoCaptureSupported` **[NEW]**
- `AVCapturePhotoOutput.isLivePhotoCaptureEnabled` **[NEW]**
- `AVCapturePhotoOutput.isLivePhotoCaptureSuspended` **[NEW]**
- `AVCapturePhotoOutput.isHighResolutionCaptureEnabled` **[NEW]**
- `AVCapturePhotoOutput.availableRawPhotoPixelFormatTypes` **[NEW]**
- `AVCapturePhotoOutput.dngPhotoDataRepresentation(_:previewPhotoSampleBuffer:)` **[NEW]** — writes compressed DNG
- `AVCapturePhotoOutput.jpegPhotoDataRepresentation(forJPEGSampleBuffer:previewPhotoSampleBuffer:)` **[NEW]**
- `AVCapturePhotoSettings.isHighResolutionPhotoEnabled`
- `AVCapturePhotoSettings.flashMode`
- `AVCapturePhotoSettings.isAutoStillImageStabilizationEnabled`
- `AVCapturePhotoSettings.livePhotoMovieFileURL`
- `AVCapturePhotoSettings.previewPhotoFormat` — dictionary with `kCVPixelBufferPixelFormatTypeKey`, optional width/height
- `AVCaptureDevice.Format.supportedColorSpaces` **[NEW]**
- `AVCaptureDevice.activeColorSpace` **[NEW]** — `.sRGB` / `.P3_D65`
- `AVCaptureSession.automaticallyConfiguresCaptureDeviceForWideColor` **[NEW]**
- `AVCaptureBracketedStillImageSettings` (existing) — per-exposure settings for brackets
- `AVCaptureStillImageOutput` — **deprecated in iOS 10**
- `AVCaptureDevice` flash properties — **deprecated in iOS 10** (moved to `AVCapturePhotoSettings`)
- **Photos / PhotosUI** — `PHLivePhotoView`, Live Photo editing APIs **[NEW in iOS 10]**
- **Core Image** — new RAW processing APIs (covered in Session 505)
- `CVPixelBuffer.h` — four new Bayer-pattern RAW format constants **[NEW]**: `kCVPixelFormatType_14Bayer_GRBG`, `kCVPixelFormatType_14Bayer_RGGB`, `kCVPixelFormatType_14Bayer_BGGR`, `kCVPixelFormatType_14Bayer_GBRG`

## Code Highlights

Basic photo capture with the new output:
```swift
let settings = AVCapturePhotoSettings()
settings.isHighResolutionPhotoEnabled = true
photoOutput.capturePhoto(with: settings, delegate: self)
```

Requesting a RAW + JPEG capture:
```swift
let rawFormat = photoOutput.availableRawPhotoPixelFormatTypes.first!
let settings = AVCapturePhotoSettings(rawPixelFormatType: rawFormat,
                                       processedFormat: [AVVideoCodecKey: AVVideoCodecJPEG])
```

Storing a RAW buffer as a DNG with embedded thumbnail:
```swift
func photoOutput(_ output: AVCapturePhotoOutput,
                 didFinishProcessingRawPhoto rawSampleBuffer: CMSampleBuffer?,
                 previewPhoto previewSampleBuffer: CMSampleBuffer?, ...) {
    guard let raw = rawSampleBuffer else { return }
    let dngData = AVCapturePhotoOutput.dngPhotoDataRepresentation(
        forRawSampleBuffer: raw, previewPhotoSampleBuffer: previewSampleBuffer)
    try? dngData?.write(to: outputURL)
}
```

## Takeaways
- Migrate from `AVCaptureStillImageOutput` to `AVCapturePhotoOutput` immediately; the old class is deprecated in iOS 10 and the new API is cleaner, safer, and extensible.
- Live Photo capture requires opting in before `startRunning()`, a microphone input, and the photo preset; movies are now automatically stabilized in iOS 10.
- RAW photo capture is now supported on iOS for the first time (devices with 12 MP cameras); always embed a preview thumbnail when writing DNG files to the photo library.
- Wide-color (Display P3) photo capture on iPad Pro 9.7 is automatic when using `AVCapturePhotoOutput`; avoid forcing P3 on video outputs unless your entire pipeline is color-aware.

---
_Source: WWDC16 Session 501 page (abstract, transcript, and resource links)._
