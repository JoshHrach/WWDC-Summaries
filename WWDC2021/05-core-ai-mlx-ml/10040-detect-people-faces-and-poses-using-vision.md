# Detect people, faces, and poses using Vision
**WWDC21 · Session 10040** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10040/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session covers the iOS 15 / macOS Monterey updates to the Vision framework's people analysis capabilities, culminating in the introduction of the new Person Segmentation API. Existing face, body, and hand detection features are upgraded with new revisions, and a brand-new `VNGeneratePersonSegmentationRequest` is introduced for semantically separating people from their backgrounds in both streaming and offline contexts.

The session also surveys the equivalent person segmentation APIs available in AVFoundation, ARKit, and Core Image, providing guidance on which API to choose based on the application context. All of this is illustrated with live demos—including one combining face pose detection with real-time background replacement using the new segmentation API.

## Key Topics

### Face Analysis Upgrades

#### Face Detection — Revision 3 **[NEW]**
- `VNDetectFaceRectanglesRequest` revision 3 adds detection of faces covered by masks (previously only handled glasses and hat occlusions).
- Improved precision and recall across arbitrary orientations, sizes, and partial occlusions.

#### Face Pose Metrics — Continuous and Complete **[NEW]**
- Previously, roll and yaw only, reported in discrete bins.
- Revision 3 adds **pitch** (nodding up/down) to complete the full roll/yaw/pitch triad.
- All three metrics are now reported as **continuous floating-point values** in radians (previously binned).
- Returned via `VNFaceObservation.pitch`, `.yaw`, `.roll` as optional `NSNumber` values.

#### Face Landmarks — Revision 3 (existing)
- `VNDetectFaceLandmarksRequest` revision 3: 76-point constellation, accurate pupil detection.

#### Face Capture Quality — Revision 2 (existing)
- `VNDetectFaceCaptureQualityRequest` revision 2: holistic quality measure (lighting, expression, occlusion, blur, focus).
- Comparative measure for the same subject (e.g., best photo in a burst); not for cross-person comparison.

### Body and Hand Analysis Upgrades

#### Human Body Detection — Revision 2 **[NEW]**
- `VNDetectHumanRectanglesRequest` revision 2 adds **full-body detection** in addition to existing upper-body detection.
- New `upperBodyOnly: Bool` property (default `true` for backward compatibility).

#### Hand Chirality **[NEW]**
- `VNHumanHandPoseObservation` gains a new `chirality` property indicating whether the detected hand is left or right.
- Works with existing `VNDetectHumanHandPoseRequest`.

### Person Segmentation API (Vision) **[NEW]**

#### Overview
- Semantic segmentation: produces a single mask for all people in the frame.
- Designed for both streaming pipelines (stateful, reuses request object for temporal smoothing) and offline single-frame processing.
- Supported: iOS 15, iPadOS 15, macOS Monterey, tvOS 15.

#### `VNGeneratePersonSegmentationRequest`
- `revision`: `VNGeneratePersonSegmentationRequestRevision1` (set explicitly for determinism).
- `qualityLevel`:
  - `.accurate` (default) — highest quality, recommended for computational photography.
  - `.balanced` — recommended for frame-by-frame video segmentation.
  - `.fast` — recommended for live streaming; temporal smoothing applied.
- `outputPixelFormat`:
  - `kCVPixelFormatType_OneComponent8` — 8-bit unsigned integer, 0–255.
  - `kCVPixelFormatType_OneComponent32Float` — 32-bit float.
  - `kCVPixelFormatType_OneComponent16Half` — 16-bit half-precision float (optimized for Metal GPU pipelines).

#### Result
- Returns `VNPixelBufferObservation` with a `pixelBuffer: CVPixelBuffer` property.
- Higher quality levels: finer detail, higher resolution mask, more memory and compute.

#### Best Practices
- Up to 4 people in frame; subjects mostly visible (natural occlusions OK).
- Each person's height should be at least half the image height; good contrast with background.
- Avoid scenes with statues, photos of people, or subjects at far distance.

### Applying Segmentation Masks with Core Image
- Scale mask and background to input image dimensions using `CIImage.transformed(by:)`.
- Blend with `CIFilter.blendWithRedMask()` (single-component pixel buffers map to the red channel by default).

### Person Segmentation in Other Frameworks
- **AVFoundation**: `AVCapturePhoto.portraitEffectsMatte` (check `isPortraitEffectsMatteDeliverySupported`, enable delivery) — some newer devices during capture sessions.
- **ARKit**: `ARFrame.segmentationBuffer` (check `ARWorldTrackingConfiguration.supportsFrameSemantics(.personSegmentationWithDepth)`) — A12 Bionic and later.
- **Core Image**: `CIFilter.personSegmentation()` — thin wrapper around Vision; stays within the CI domain.

## APIs & Frameworks

**Vision**
- `VNDetectFaceRectanglesRequest` revision 3 — mask occlusion support, continuous pose metrics **[NEW]**
- `VNFaceObservation.pitch` — new continuous pitch metric **[NEW]**
- `VNFaceObservation.yaw` — now continuous (was binned) **[UPDATED]**
- `VNFaceObservation.roll` — now continuous (was binned) **[UPDATED]**
- `VNDetectFaceLandmarksRequest` revision 3 — 76-point constellation **[existing]**
- `VNDetectFaceCaptureQualityRequest` revision 2 **[existing]**
- `VNDetectHumanRectanglesRequest` revision 2 — full-body detection **[NEW]**
- `VNDetectHumanRectanglesRequest.upperBodyOnly` — `Bool` property, default `true` **[NEW]**
- `VNDetectHumanBodyPoseRequest` revision 1 **[existing]**
- `VNDetectHumanHandPoseRequest` **[existing]**
- `VNHumanHandPoseObservation.chirality` — `.left` / `.right` **[NEW]**
- `VNGeneratePersonSegmentationRequest` — new stateful segmentation request **[NEW]**
- `VNGeneratePersonSegmentationRequest.QualityLevel` enum: `.accurate`, `.balanced`, `.fast` **[NEW]**
- `VNGeneratePersonSegmentationRequest.outputPixelFormat` **[NEW]**
- `VNPixelBufferObservation` — result type with `pixelBuffer` **[existing, returned by new request]**
- `VNImageRequestHandler` — processes requests against images or video frames **[existing]**

**Core Image**
- `CIFilter.personSegmentation()` — wraps Vision segmentation **[NEW]**
- `CIFilter.blendWithRedMask()` — composites input, background, and single-channel mask **[existing]**

**AVFoundation**
- `AVCapturePhotoOutput.isPortraitEffectsMatteDeliverySupported` / `.isPortraitEffectsMatteDeliveryEnabled` **[existing]**
- `AVCapturePhoto.portraitEffectsMatte` — segmentation mask from capture **[existing]**

**ARKit**
- `ARWorldTrackingConfiguration.supportsFrameSemantics(_:)` with `.personSegmentationWithDepth` **[existing]**
- `ARFrame.segmentationBuffer: CVPixelBuffer?` **[existing]**

## Code Highlights

Basic person segmentation request:
```swift
let request = VNGeneratePersonSegmentationRequest()
let requestHandler = VNImageRequestHandler(url: imageURL, options: [:])
try requestHandler.perform([request])
let mask = request.results!.first!
let maskBuffer = mask.pixelBuffer
```

Configuring quality and output format:
```swift
request.revision = VNGeneratePersonSegmentationRequestRevision1
request.qualityLevel = .accurate
request.outputPixelFormat = kCVPixelFormatType_OneComponent8
```

Applying mask with Core Image blend filter:
```swift
let blendFilter = CIFilter.blendWithRedMask()
blendFilter.inputImage = input
blendFilter.backgroundImage = backgroundScaled
blendFilter.maskImage = maskScaled
let blendedImage = blendFilter.outputImage
```

Core Image convenience API:
```swift
let segmentationFilter = CIFilter.personSegmentation()
segmentationFilter.inputImage = input
let mask = segmentationFilter.outputImage
```

## Takeaways
- Vision now supports mask-occluded face detection (revision 3) and full-body detection (revision 2), plus continuous roll/yaw/pitch and new hand chirality.
- `VNGeneratePersonSegmentationRequest` is a stateful request with three quality levels; choose `.fast` for streaming, `.balanced` for video, `.accurate` for photography.
- Pick the right segmentation API for your context: AVFoundation for capture sessions, ARKit for AR apps, Vision or Core Image for general image/video processing.
- Use `CIFilter.blendWithRedMask()` to composite the segmentation mask in the Core Image pipeline with minimal code.

---
_Source: WWDC21 Session 10040 page (abstract, full transcript, code samples, and resource links)._
