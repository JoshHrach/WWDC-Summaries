# Discover Advancements in iOS Camera Capture: Depth, Focus, and Multitasking
**WWDC22 · Session 110429** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110429/)

_Platforms:_ iOS 16, iPadOS 16

## Overview
This session covers four significant new camera capture capabilities introduced in iOS 16 and iPadOS 16. The first is streaming depth data from LiDAR Scanners via AVFoundation using the new `builtInLiDARDepthCamera` device type — previously only available through ARKit. The second is face-driven autofocus and autoexposure improvements that are enabled by default for all apps linked on iOS 15.4 or later. The third is support for multiple simultaneous `AVCaptureVideoDataOutput` instances, each independently configurable for resolution, stabilization, orientation, and pixel format. The fourth is the ability for capture sessions to remain active while the user multitasks on iPad, including Stage Manager support.

## Key Topics

### LiDAR Depth Camera in AVFoundation
- New `AVCaptureDevice.DeviceType.builtInLiDARDepthCamera` **[NEW in iOS 15.4]** — streams video and high-accuracy absolute depth simultaneously.
- Uses the rear wide-angle camera combined with LiDAR Scanner; depth processed via ML fusion for dense depth maps.
- Video resolutions from 640×480 up to full 12MP (4032×3024); depth maps up to 320×240 streaming, 768×576 for photos.
- Compatible with `AVCaptureMultiCamSession` alongside Telephoto and Ultra Wide cameras.
- Available on iPhone 12 Pro, iPhone 13 Pro, iPad Pro 5th generation.
- Produces absolute (real-world scale) depth, unlike TrueDepth/Dual/Triple which produce relative disparity-based depth.
- `AVDepthData` class: carries depth pixel buffer, depth data type, accuracy, and filtered flag.
- `AVCaptureDepthDataOutput` — streams depth frames; `AVCapturePhotoOutput` — attaches depth to photos.
- `AVCaptureDepthDataOutput.isFilteringEnabled` — set to `false` for computer vision tasks to preserve raw values; default `true` for photography/video apps.

### Face-Driven Autofocus and Autoexposure
- `isFaceDetectionAutoFocusEnabled` / `isFaceDetectionAutoExposureEnabled` **[NEW in iOS 15.4]** — enabled by default for apps linked on iOS 15.4+.
- `automaticallyAdjustsFaceDetectionAutoFocusEnabled` / `automaticallyAdjustsFaceDetectionAutoExposureEnabled` — control whether system manages these automatically.
- To opt out: lock device for configuration, disable automatic adjustment, then disable face-driven focus/exposure, unlock.
- Great for video conferencing (FaceTime), photography apps; inappropriate for apps that give users manual camera control.

### Multiple AVCaptureVideoDataOutputs
- **[NEW in iOS 16]** — apps can now add multiple `AVCaptureVideoDataOutput` instances to the same session simultaneously.
- Each output independently configurable for: resolution, stabilization mode, orientation, pixel format.
- Key pattern: one output for low-latency preview (small buffers, no stabilization), one for high-quality recording (4K, stabilization).
- Configuration options:
  - `videoSettings[kCVPixelBufferWidthKey/HeightKey]` — specify custom resolution (aspect ratio must match source format).
  - `automaticallyConfiguresOutputBufferDimensions = false` — disable automatic resolution config.
  - `deliversPreviewSizedOutputBuffers = true/false` — request preview-sized buffers.
  - `preferredVideoStabilizationMode` — set to `.cinematicExtended` for recording output.
  - Pixel format: supports 10-bit lossless YUV buffers.
- See TN3121 for pixel format selection guidance.

### Combined Movie File Output with Data Outputs
- **[NEW in iOS 16]** — `AVCaptureMovieFileOutput` can now be used simultaneously with `AVCaptureVideoDataOutput` and `AVCaptureAudioDataOutput`.
- `AVCaptureSession.canAddOutput(_:)` — check before adding.
- `AVCaptureSession.hardwareCost` — query system support for current configuration.
- Use case: offload recording mechanics to `AVCaptureMovieFileOutput` while still receiving raw frames for analysis.

### Camera Access While Multitasking
- **[NEW in iOS 16]** — `AVCaptureSession.isMultitaskingCameraAccessSupported` and `isMultitaskingCameraAccessEnabled` **[NEW]**.
- Sessions with multitasking enabled no longer interrupted with `.videoDeviceNotAvailableWithMultipleForegroundApps`.
- System shows a one-time dialog after recording informing the user of potential lower quality when multitasking.
- Supports Split View, Slide Over, and Stage Manager (iPadOS 16).
- Apps can still opt out to require full-screen camera experience (e.g., ARKit does not support this).
- Monitor `AVCaptureSession` pressure notifications; reduce frame rate or request lower-resolution/non-HDR formats to maintain performance.
- Video calling apps: use `AVPictureInPictureVideoCallViewController` (AVKit, iOS 15) to display remote participants in system PiP window during multitasking.

## APIs & Frameworks

### AVFoundation
- `AVCaptureDevice.DeviceType.builtInLiDARDepthCamera` **[NEW]**
- `AVCaptureDepthDataOutput` — stream depth frames
- `AVCaptureDepthDataOutput.isFilteringEnabled` — control depth filtering
- `AVDepthData` — depth data container (pixelBuffer, depthDataType, depthDataAccuracy, isDepthDataFiltered)
- `AVCaptureDevice.isFaceDetectionAutoFocusEnabled` **[NEW]**
- `AVCaptureDevice.isFaceDetectionAutoExposureEnabled` **[NEW]**
- `AVCaptureDevice.automaticallyAdjustsFaceDetectionAutoFocusEnabled` **[NEW]**
- `AVCaptureDevice.automaticallyAdjustsFaceDetectionAutoExposureEnabled` **[NEW]**
- `AVCaptureVideoDataOutput` — multiple instances supported per session **[NEW behavior]**
- `AVCaptureVideoDataOutput.automaticallyConfiguresOutputBufferDimensions`
- `AVCaptureVideoDataOutput.deliversPreviewSizedOutputBuffers`
- `AVCaptureVideoDataOutput.videoSettings` — kCVPixelBufferWidthKey, kCVPixelBufferHeightKey
- `AVCaptureConnection.preferredVideoStabilizationMode`
- `AVCaptureMovieFileOutput` — concurrent use with video/audio data outputs **[NEW]**
- `AVCaptureSession.canAddOutput(_:)`
- `AVCaptureSession.hardwareCost`
- `AVCaptureSession.isMultitaskingCameraAccessSupported` **[NEW]**
- `AVCaptureSession.isMultitaskingCameraAccessEnabled` **[NEW]**
- `AVCaptureMultiCamSession` — multi-camera sessions
- `AVCapturePhotoOutput` — depth data attached to photos

### AVKit
- `AVPictureInPictureVideoCallViewController` — PiP for video call remote participants (iOS 15)

## Code Highlights

```swift
// Opt out of face-driven autofocus
try device.lockForConfiguration()
device.automaticallyAdjustsFaceDetectionAutoFocusEnabled = false
device.isFaceDetectionAutoFocusEnabled = false
device.unlockForConfiguration()

// Enable camera while multitasking
session.isMultitaskingCameraAccessEnabled = true

// Add second video data output for preview
let previewOutput = AVCaptureVideoDataOutput()
previewOutput.automaticallyConfiguresOutputBufferDimensions = false
previewOutput.deliversPreviewSizedOutputBuffers = true
session.addOutput(previewOutput)

// Add full-resolution output for recording
let recordOutput = AVCaptureVideoDataOutput()
recordOutput.automaticallyConfiguresOutputBufferDimensions = false
recordOutput.deliversPreviewSizedOutputBuffers = false
// Set stabilization on the connection
recordOutput.connection(with: .video)?.preferredVideoStabilizationMode = .cinematicExtended
session.addOutput(recordOutput)
```

## Takeaways
- The new `builtInLiDARDepthCamera` brings LiDAR depth to AVFoundation for the first time, enabling high-resolution depth maps in photo/video apps without requiring ARKit.
- Face-driven AF/AE is on by default in iOS 15.4+; apps that need manual camera control should explicitly opt out.
- Multiple `AVCaptureVideoDataOutput` instances in iOS 16 eliminate the tradeoff between preview quality and recording quality by allowing separate configurations for each.
- Multitasking camera access (iOS 16) enables split-view and Stage Manager scenarios; apps must handle system pressure proactively and can opt out if full-screen is required.

---
_Source: WWDC22 Session 110429 page (abstract, chapter summaries, code samples, and resource links)._
