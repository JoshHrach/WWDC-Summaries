# What's new in Vision
**WWDC22 · Session 10024** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10024/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
The Vision framework received three new algorithm revisions in 2022, each leveraging updated machine learning models: a new text recognition revision (Revision 3) adding Korean and Japanese support with automatic language detection; a new barcode detection revision (Revision 3) using ML for improved multi-code detection, better bounding boxes, and fewer duplicates; and a new ML-based optical flow revision (Revision 2) with dramatically improved accuracy for motion estimation in video.

A notable housekeeping change: Revision 1 of face detection and face landmarks is being deprecated and removed from the framework. Code compiled against old SDKs or explicitly requesting Revision 1 will automatically be satisfied by the Revision 2 detector internally, preserving behavioral compatibility. This allows Apple to ship a leaner framework.

Xcode 14 adds Quick Look Preview support for Vision observations in the debugger and in Xcode Playgrounds, letting developers visualize detection results overlaid on input images with a single click.

## Key Topics

### Text Recognition — Revision 3
`VNRecognizeTextRequestRevision3` powers Live Text. Adds Korean and Japanese language support. New `automaticallyDetectsLanguage` property (accurate mode only) for cases where the script language is unknown at request time. `supportedRecognitionLanguages` reports the full set of supported languages.

### Barcode Detection — Revision 3
`VNDetectBarcodesRequestRevision3` is ML-based for the first time. Key improvements: simultaneous multi-code detection (faster for images with many codes), fewer duplicate detections, complete bounding boxes for linear codes (previously only a centerline), better handling of curved surfaces and reflections. `supportedSymbologies` enumerates all supported barcode types. Powers the new VisionKit Data Scanner API.

### Optical Flow — Revision 2
`VNGenerateOpticalFlowRequestRevision2` is ML-based, replacing the classical algorithm of Revision 1. Returns a `VNPixelBufferObservation` — a two-channel image encoding X and Y motion vectors at each pixel. Improvements: more accurate motion capture for objects (e.g., animal body parts), cleaner background (less false motion), better handling of small objects. Underlying model output is lower resolution than input; Vision upsamples by default using bilinear interpolation. `keepNetworkOutput` property skips upsampling for raw model output. `computationAccuracy` property controls output resolution (four levels).

### Face Detection / Landmarks — Revision 1 Deprecation
`VNDetectFaceRectanglesRequestRevision1` and `VNDetectFaceLandmarksRequestRevision1` are deprecated and removed from the framework binary, but Revision 1 behavior is preserved: the system internally routes Revision 1 requests through the Revision 2 detector while suppressing upside-down faces and returning Revision 1 landmark constellation format. No code changes required, but migration to Revision 3 is strongly recommended.

### Quick Look Preview in Xcode
`VNObservation` and its subclasses now support Quick Look in Xcode debugger (hover + click the eye icon) and in Xcode Playgrounds. Shows bounding boxes, landmarks, and other result overlays on the original input image. Requires the `VNImageRequestHandler` to remain in scope (it holds the input image).

## APIs & Frameworks

**Vision**
- `VNRecognizeTextRequestRevision3` **[NEW]** — updated text recognition model
  - `VNRecognizeTextRequest.automaticallyDetectsLanguage` **[NEW]** — auto language detection (accurate mode)
  - `VNRecognizeTextRequest.recognitionLanguages` — explicit language list
  - `VNRecognizeTextRequest.supportedRecognitionLanguages()` — all supported languages
- `VNDetectBarcodesRequestRevision3` **[NEW]** — ML-based barcode detection
  - `VNDetectBarcodesRequest.supportedSymbologies` — available barcode types
  - `VNBarcodeObservation` — barcode result with full bounding box
- `VNGenerateOpticalFlowRequestRevision2` **[NEW]** — ML-based optical flow
  - `VNGenerateOpticalFlowRequest.keepNetworkOutput` **[NEW]** — skip upsampling, return raw model output
  - `VNGenerateOpticalFlowRequest.computationAccuracy` — resolution/speed tradeoff (`.low`, `.medium`, `.high`, `.veryHigh`)
  - `VNPixelBufferObservation` — two-channel (X, Y) motion vector image result
- `VNDetectFaceRectanglesRequestRevision1` — deprecated (removed from binary, rerouted to Rev 2)
- `VNDetectFaceLandmarksRequestRevision1` — deprecated (removed from binary, rerouted to Rev 2)
- `VNDetectFaceRectanglesRequestRevision3` — recommended current revision
- `VNDetectFaceLandmarksRequestRevision3` — recommended current revision
- `VNImageRequestHandler` — must be in scope for Quick Look Preview to display input image
- `VNRequest` — base request class
- `VNObservation` — base observation with Quick Look support **[NEW]**
- `VNFaceObservation` — face detection result
- `VNRecognizedTextObservation` — text recognition result
- `VNBarcodeObservation` — barcode detection result

## Code Highlights

Using Revision 3 text recognition with automatic language detection:
```swift
let request = VNRecognizeTextRequest()
request.revision = VNRecognizeTextRequestRevision3
request.recognitionLevel = .accurate
request.automaticallyDetectsLanguage = true  // NEW

let handler = VNImageRequestHandler(cgImage: cgImage)
try handler.perform([request])
let results = request.results as? [VNRecognizedTextObservation]
```

Using Revision 2 optical flow with raw network output:
```swift
let request = VNGenerateOpticalFlowRequest(
    targetedCVPixelBuffer: nextFrame)
request.revision = VNGenerateOpticalFlowRequestRevision2
request.computationAccuracy = .high
request.keepNetworkOutput = true   // skip upsampling

let handler = VNImageRequestHandler(cvPixelBuffer: currentFrame)
try handler.perform([request])
let flowObservation = request.results?.first as? VNPixelBufferObservation
// flowObservation.pixelBuffer contains 2-channel X/Y vectors
```

## Takeaways
- Use `VNRecognizeTextRequestRevision3` for the best OCR performance, including new Korean/Japanese support and automatic language detection.
- `VNDetectBarcodesRequestRevision3` replaces classical barcode detection with an ML model that handles multiple codes simultaneously with complete bounding boxes.
- `VNGenerateOpticalFlowRequestRevision2`'s ML model produces significantly cleaner motion estimates; use `keepNetworkOutput = true` to skip bilinear upsampling when raw lower-resolution vectors are sufficient.
- Always explicitly specify revision numbers in Vision requests; Revision 1 of face APIs is deprecated — migrate to Revision 3 for best accuracy.

---
_Source: WWDC22 Session 10024 page (abstract, chapter summaries, code samples, and resource links)._
