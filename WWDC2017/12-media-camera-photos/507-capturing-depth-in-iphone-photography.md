# Capturing Depth in iPhone Photography
**WWDC17 · Session 507** · [Watch](https://developer.apple.com/videos/play/wwdc2017/507/)

_Platforms:_ iOS 11, macOS High Sierra 10.13, tvOS 11

## Overview
iOS 11 opens up the iPhone 7 Plus dual-camera depth maps — previously exclusive to Portrait mode — to third-party apps via a new set of AVFoundation APIs. This session (part one of a two-part series) covers the underlying concepts of depth and disparity, then walks through the full capture pipeline: streaming depth in real time from the camera, capturing still photos with embedded depth data, obtaining camera calibration data, and the new dual-photo capture feature.

The presenter explains why the dual-camera system generates disparity (a proxy for depth, measured as the pixel shift of the same point seen from two different cameras) rather than true time-of-flight depth. Normalized disparity — expressed as inverse meters rather than pixel offsets — is the canonical representation because it survives image scaling. `AVDepthData` is the platform object for all depth/disparity data and can represent both absolute and relative (the iPhone case) accuracy modes.

Two demo apps are shown: `AVCamPhotoFilter` (streaming depth preview with real-time smoothing/hole-filling) and `AVCam` (capturing full-res photos with embedded depth), plus `Wiggle Me` (creative 3D parallax effects from depth-embedded photos) and `Straighten Up` (lens distortion correction using `AVCameraCalibrationData`).

## Key Topics

- **Depth vs. disparity** — dual camera is a disparity-based stereo system, not time-of-flight; normalized disparity is 1/Z, resilient to scaling; relative vs. absolute accuracy.
- **AVDepthData** — the platform's canonical depth/disparity container; backed by a `CVPixelBuffer`; knows whether data is filtered (holes filled) or raw; carries accuracy metadata.
- **Four new CoreVideo pixel formats** — `kCVPixelFormatType_DisparityFloat16`, `kCVPixelFormatType_DisparityFloat32`, `kCVPixelFormatType_DepthFloat16`, `kCVPixelFormatType_DepthFloat32`.
- **Holes in depth data** — occur at occlusions and textureless regions; represented as NaN; `isDepthDataFiltered` indicates whether holes have been interpolated.
- **AVCaptureDepthDataOutput (DDO)** — new output class delivering streaming `AVDepthData` at 15–24 fps; requires iPhone 7 Plus dual camera; locks zoom to 2X; supports three formats (Photo, 16:9 30fps, VGA).
- **AVCaptureDataOutputSynchronizer** — new synchronizer that delivers video, audio, metadata, and depth in a single unified callback keyed to a presentation timestamp; eliminates complex multi-buffer coordination.
- **Camera intrinsics delivery** — opt-in per connection; delivers a 3×3 `matrix_float3x3` attachment with each video frame containing focal length (fx, fy) and optical center (x₀, y₀); used by ARKit.
- **AVCapturePhotoOutput + depth** — implementing the new `AVCapturePhotoCaptureDelegate` callback receiving `AVCapturePhoto`; opting in via `isDepthDataDeliveryEnabled`; depth stored as auxiliary image in HEIF or as MPO in JPEG.
- **HEIF (HEIC) depth storage** — depth encoded as monochrome HEVC auxiliary image with XMP metadata including accuracy, calibration, and rendering instructions.
- **Dual photo capture** — new opt-in feature returning both the wide (28mm equivalent) and tele (56mm equivalent) images at full 12 MP from a single `AVCapturePhotoSettings` request; field-of-view black masking applied when zoomed in.
- **AVCameraCalibrationData** — provides `intrinsicMatrix`, `extrinsicMatrix`, `lensDistortionCenter`, `lensDistortionLookupTable`, and `inverseLensDistortionLookupTable` for lens distortion correction and 3D reprojection.
- **Lens distortion correction** — depth maps shipped in photos are warped to match the RGB image; use `lensDistortionLookupTable` to make them rectilinear; reference implementation is in `AVCameraCalibrationData.h`.

## APIs & Frameworks

**AVFoundation**
- `AVDepthData` **[NEW]** — canonical container for depth/disparity maps
- `AVDepthData.depthDataMap` **[NEW]** — `CVPixelBuffer` backing store
- `AVDepthData.depthDataType` **[NEW]** — one of four new CoreVideo formats
- `AVDepthData.isDepthDataFiltered` **[NEW]** — whether holes have been filled
- `AVDepthData.depthDataAccuracy` **[NEW]** — `.absolute` or `.relative`
- `AVDepthData.converting(toDepthDataType:)` **[NEW]** — convert between depth and disparity formats
- `AVCaptureDepthDataOutput` **[NEW]** — streaming depth output attached to an `AVCaptureSession`
- `AVCaptureDepthDataOutputDelegate` **[NEW]** — delivers `AVDepthData` objects per frame
- `AVCaptureDepthDataOutput.isFilteringEnabled` **[NEW]** — enables hole-filling and temporal smoothing
- `AVCaptureDevice.supportedDepthDataFormats` **[NEW]** — depth-capable formats on dual camera
- `AVCaptureDevice.activeDepthDataFormat` **[NEW]** — get/set active depth format
- `AVCaptureDataOutputSynchronizer` **[NEW]** — synchronizes multiple outputs to one callback
- `AVCaptureSynchronizedDataCollection` **[NEW]** — collection of synchronized outputs for one timestamp
- `AVCaptureSynchronizedDepthData` **[NEW]** — depth result in a synchronized collection
- `AVCaptureConnection.isCameraIntrinsicMatrixDeliverySupported` **[NEW]**
- `AVCaptureConnection.isCameraIntrinsicMatrixDeliveryEnabled` **[NEW]** — opt-in for intrinsics per frame
- `AVCapturePhotoOutput.isDepthDataDeliverySupported` **[NEW]**
- `AVCapturePhotoOutput.isDepthDataDeliveryEnabled` **[NEW]** — session-level opt-in for depth in photos
- `AVCapturePhotoSettings.isDepthDataDeliveryEnabled` **[NEW]** — per-photo opt-in
- `AVCapturePhoto` **[NEW]** — new unified photo result object (replaces sample buffer callback)
- `AVCapturePhoto.depthData` **[NEW]** — `AVDepthData` embedded in the captured photo
- `AVCapturePhotoOutput.isDualCameraFusionSupported` / `AVCapturePhotoSettings.isDualCameraFusionEnabled` **[NEW]** — dual photo capture opt-in
- `AVCameraCalibrationData` **[NEW]** — camera calibration model
- `AVCameraCalibrationData.intrinsicMatrix` **[NEW]** — 3×3 `matrix_float3x3`
- `AVCameraCalibrationData.intrinsicMatrixReferenceDimensions` **[NEW]`
- `AVCameraCalibrationData.extrinsicMatrix` **[NEW]** — 3×4 rotation + translation matrix
- `AVCameraCalibrationData.lensDistortionCenter` **[NEW]** — `CGPoint`
- `AVCameraCalibrationData.lensDistortionLookupTable` **[NEW]** — `Data` wrapping `[Float]`
- `AVCameraCalibrationData.inverseLensDistortionLookupTable` **[NEW]`

**CoreVideo**
- `kCVPixelFormatType_DisparityFloat16` **[NEW]**
- `kCVPixelFormatType_DisparityFloat32` **[NEW]**
- `kCVPixelFormatType_DepthFloat16` **[NEW]**
- `kCVPixelFormatType_DepthFloat32` **[NEW]**

**Sample Code**
- `AVCamPhotoFilter` — streaming depth preview + photo with depth
- `AVCam` — standard camera app updated for depth capture
- `Using Depth Data` — sample demonstrating `AVDepthData` usage

## Code Highlights

Attaching depth output and synchronizer:
```swift
let depthOutput = AVCaptureDepthDataOutput()
depthOutput.isFilteringEnabled = true
session.addOutput(depthOutput)

let synchronizer = AVCaptureDataOutputSynchronizer(dataOutputs: [videoOutput, depthOutput])
synchronizer.setDelegate(self, queue: sessionQueue)
```

Handling synchronized callback:
```swift
func dataOutputSynchronizer(_ synchronizer: AVCaptureDataOutputSynchronizer,
    didOutput collection: AVCaptureSynchronizedDataCollection) {
    guard let depthData = collection[depthOutput] as? AVCaptureSynchronizedDepthData,
          !depthData.depthDataWasDropped else { return }
    let depth = depthData.depthData  // AVDepthData
}
```

Opting in to depth in photos:
```swift
photoOutput.isDepthDataDeliveryEnabled = true

let settings = AVCapturePhotoSettings()
settings.isDepthDataDeliveryEnabled = true
photoOutput.capturePhoto(with: settings, delegate: self)

// In delegate:
func photoOutput(_ output: AVCapturePhotoOutput,
    didFinishProcessingPhoto photo: AVCapturePhoto, error: Error?) {
    let depthData = photo.depthData  // AVDepthData?
}
```

## Takeaways

- iPhone 7 Plus depth maps (disparity, relative accuracy) are now available to third-party apps via `AVCaptureDepthDataOutput` for streaming and `AVCapturePhotoOutput` for stills.
- Use `AVCaptureDataOutputSynchronizer` to receive video, audio, metadata, and depth in one unified callback rather than managing four independent queues.
- Depth maps in captured photos are lens-distorted to match the RGB image; use `AVCameraCalibrationData.lensDistortionLookupTable` to make them rectilinear for computer-vision or AR tasks.
- Dual photo capture delivers both 12 MP wide and tele images from a single request, enabling custom depth reconstruction, AR, and computational photography pipelines.

---
_Source: WWDC17 Session 507 page (abstract, transcript, and resource links)._
