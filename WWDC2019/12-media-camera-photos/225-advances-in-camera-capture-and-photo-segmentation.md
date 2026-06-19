# Advances in Camera Capture & Photo Segmentation
**WWDC19 · Session 225** · [Watch](https://developer.apple.com/videos/play/wwdc2019/225/)

_Platforms:_ iOS 13, iPadOS 13

## Overview
iOS 13 delivers two major advances in camera and photo APIs. First, the new AVCaptureMultiCamSession enables simultaneous capture from multiple cameras on the same device — the front camera and rear camera running at the same time, or both rear cameras — unlocking picture-in-picture recording, real-time multi-angle capture, and similar experiences that were previously impossible. Second, AVFoundation introduces AVSemanticSegmentationMatte, a framework-level abstraction that extends portrait-segmentation mattes beyond the existing person/portrait matte to cover specific semantic regions: hair, skin, and teeth. These new mattes make it straightforward to build per-region photo editing effects without custom ML pipelines.

The session also covers improvements to the virtual camera device model introduced for the triple-camera iPhone 11 lineup, including the ultra-wide lens, geometric distortion correction for that lens, and auto-switching across constituent devices as the user changes zoom level.

## Key Topics

### AVCaptureMultiCamSession **[NEW]**
- `AVCaptureMultiCamSession` — new subclass of `AVCaptureSession` that allows adding inputs from more than one camera simultaneously.
- Enables simultaneous front + rear camera capture, or wide + telephoto, on supported hardware.
- Each camera's output flows to its own independent set of `AVCaptureOutput` objects (video data output, photo output, movie file output, etc.).
- Hardware limits apply: not all device/camera combinations support multi-cam simultaneously; check `AVCaptureMultiCamSession.isMultiCamSupported` and the `AVCaptureDeviceInput.isPortOwner` state.
- CPU and thermal cost is higher; monitor `AVCaptureSession.hardwareCost` and `systemPressureState`.

### Virtual Camera Devices & Ultra-Wide **[NEW]**
- `AVCaptureDevice.DeviceType.builtInDualWideCamera` **[NEW]** — ultra-wide + wide virtual device.
- `AVCaptureDevice.DeviceType.builtInTripleCamera` **[NEW]** — ultra-wide + wide + telephoto virtual device.
- `AVCaptureDevice.DeviceType.builtInUltraWideCamera` **[NEW]** — ultra-wide camera directly.
- Virtual devices auto-switch between constituent physical cameras as `videoZoomFactor` changes, providing seamless optical zoom across the entire range.
- `AVCaptureDevice.primaryConstituentDevice` **[NEW]** — the currently active physical camera within a virtual device.
- `AVCaptureDevice.constituentDevices` — array of all physical cameras backing a virtual device.
- **Geometric distortion correction** **[NEW]**: ultra-wide lens barrel distortion is automatically corrected; controlled via `AVCaptureDevice.geometricDistortionCorrectionEnabled`.

### AVSemanticSegmentationMatte **[NEW]**
- `AVSemanticSegmentationMatte` — new class representing a per-pixel segmentation mask for a specific semantic class within a photo.
- Matte types (all **[NEW]**):
  - `.hair` — mask covering hair pixels
  - `.skin` — mask covering skin pixels
  - `.teeth` — mask covering teeth pixels
- Extends the existing `AVPortraitEffectsMatte` (person segmentation, iOS 12) with finer semantic granularity.
- Captured via `AVCapturePhotoOutput` alongside the main image and depth data; request via `AVCapturePhotoSettings.enabledSemanticSegmentationMatteTypes`.
- `AVCapturePhotoOutput.availableSemanticSegmentationMatteTypes` — query which matte types the device/configuration supports.
- Delivered in `AVCapturePhotoCaptureDelegate.photoOutput(_:didFinishProcessingPhoto:error:)` via `AVCapturePhoto.semanticSegmentationMatte(for:)`.
- Stored in HEIC/JPEG as auxiliary image (same mechanism as depth maps and portrait mattes).
- Read from the photo library via `PHAsset` + `PHImageManager` requesting `.auxiliarySemanticSegmentation*` image types, or via `CIImage` with auxiliary content.

### Photo Library Integration
- Semantic segmentation mattes are saved as auxiliary images inside HEIC containers.
- `CIImage(contentsOf:options:)` with `kCIImageAuxiliarySemanticSegmentationHairMatte`, `kCIImageAuxiliarySemanticSegmentationSkinMatte`, `kCIImageAuxiliarySemanticSegmentationTeethMatte` **[NEW]** keys to read mattes from files.
- `PHAsset` fetch with `PHImageRequestOptions` to retrieve matte auxiliary data from the photo library.

## APIs & Frameworks

**AVFoundation — Capture**
- `AVCaptureMultiCamSession` **[NEW]** — simultaneous multi-camera capture session
- `AVCaptureMultiCamSession.isMultiCamSupported: Bool` **[NEW]**
- `AVCaptureMultiCamSession.hardwareCost: Float` **[NEW]** — 0.0–1.0; must stay ≤ 1.0
- `AVCaptureMultiCamSession.systemPressureCost: Float` **[NEW]**
- `AVCaptureDevice.DeviceType.builtInUltraWideCamera` **[NEW]**
- `AVCaptureDevice.DeviceType.builtInDualWideCamera` **[NEW]**
- `AVCaptureDevice.DeviceType.builtInTripleCamera` **[NEW]**
- `AVCaptureDevice.primaryConstituentDevice: AVCaptureDevice?` **[NEW]**
- `AVCaptureDevice.constituentDevices: [AVCaptureDevice]` **[NEW]**
- `AVCaptureDevice.geometricDistortionCorrectionEnabled: Bool` **[NEW]**
- `AVCaptureDevice.isGeometricDistortionCorrectionSupported: Bool` **[NEW]**
- `AVCaptureDeviceInput` — unchanged API, works with multi-cam session

**AVFoundation — Photo Output & Segmentation Mattes**
- `AVSemanticSegmentationMatte` **[NEW]** — new class
- `AVSemanticSegmentationMatte.MatteType` **[NEW]** — `.hair`, `.skin`, `.teeth`
- `AVSemanticSegmentationMatte.mattingImage: CVPixelBuffer` **[NEW]**
- `AVSemanticSegmentationMatte(fromImageSourceAuxiliaryDataType:dictionaryRepresentation:)` **[NEW]**
- `AVCapturePhotoOutput.availableSemanticSegmentationMatteTypes: [AVSemanticSegmentationMatte.MatteType]` **[NEW]**
- `AVCapturePhotoSettings.enabledSemanticSegmentationMatteTypes: [AVSemanticSegmentationMatte.MatteType]` **[NEW]**
- `AVCapturePhoto.semanticSegmentationMatte(for:) -> AVSemanticSegmentationMatte?` **[NEW]**
- `AVPortraitEffectsMatte` — existing (iOS 12), unchanged

**Core Image**
- `CIImage` auxiliary content keys **[NEW]**: `kCIImageAuxiliarySemanticSegmentationHairMatte`, `kCIImageAuxiliarySemanticSegmentationSkinMatte`, `kCIImageAuxiliarySemanticSegmentationTeethMatte`

**Photos Framework**
- `PHAsset` + `PHImageManager` — retrieve matte auxiliary images from the library (existing mechanism, new matte types)

## Code Highlights

```swift
// Multi-cam: simultaneous front + rear capture
guard AVCaptureMultiCamSession.isMultiCamSupported else { return }
let session = AVCaptureMultiCamSession()

let frontDevice = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: .front)!
let rearDevice = AVCaptureDevice.default(.builtInWideAngleCamera, for: .video, position: .back)!

let frontInput = try AVCaptureDeviceInput(device: frontDevice)
let rearInput = try AVCaptureDeviceInput(device: rearDevice)

session.addInputWithNoConnections(frontInput)
session.addInputWithNoConnections(rearInput)
// add outputs and connections per camera...
print("Hardware cost: \(session.hardwareCost)")  // must be ≤ 1.0
```

```swift
// Semantic segmentation matte capture
let photoOutput = AVCapturePhotoOutput()
// After adding to session:
let settings = AVCapturePhotoSettings()
settings.enabledSemanticSegmentationMatteTypes = 
    photoOutput.availableSemanticSegmentationMatteTypes  // [.hair, .skin, .teeth]

// In delegate:
func photoOutput(_ output: AVCapturePhotoOutput,
                 didFinishProcessingPhoto photo: AVCapturePhoto, error: Error?) {
    let hairMatte = photo.semanticSegmentationMatte(for: .hair)
    let skinMatte = photo.semanticSegmentationMatte(for: .skin)
    let teethMatte = photo.semanticSegmentationMatte(for: .teeth)
    // use CVPixelBuffer from hairMatte?.mattingImage for compositing
}
```

```swift
// Ultra-wide virtual device with distortion correction
let device = AVCaptureDevice.default(.builtInDualWideCamera, for: .video, position: .back)!
try device.lockForConfiguration()
device.geometricDistortionCorrectionEnabled = true
device.unlockForConfiguration()
// zoom factor range automatically covers ultra-wide → wide transition
```

## Takeaways
- `AVCaptureMultiCamSession` is the most requested camera feature — it enables simultaneous front+rear recording and multi-angle apps, but requires careful hardware cost monitoring to stay within device thermal limits.
- `AVSemanticSegmentationMatte` extends the portrait matte paradigm to per-body-part masks (hair, skin, teeth), making photo effects like hair color changes or teeth whitening trivial to implement correctly.
- The triple-camera virtual device model and geometric distortion correction on ultra-wide eliminate the need for apps to manage physical camera switching or correct lens distortion manually.
- Semantic mattes are stored as auxiliary images in HEIC, so they round-trip through the Photos library and can be retrieved later — no need to store separate matte files.

---
_Source: WWDC19 Session 225 page (abstract, chapter summaries, code samples, and resource links)._
