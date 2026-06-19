# Understanding Images in Vision Framework
**WWDC19 · Session 222** · [Watch](https://developer.apple.com/videos/play/wwdc2019/222/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, tvOS 13

## Overview
Vision in iOS 13 / macOS Catalina receives a major set of additions: image saliency analysis (attention-based and objectness-based), on-device multi-label image classification using Apple's Photos search network (1000+ categories), image similarity via feature prints, a new face landmarks revision with 76 points and per-point confidence, a face capture quality scoring metric, new human and animal body detectors, a revised ML-based object tracker, and expanded multi-input/multi-output Core ML integration.

Together these features make it practical to build powerful computer vision pipelines entirely on-device. The session provides both conceptual explanations and corresponding code snippets for every new API, showing how individual features can be composed — for example, chaining objectness-based saliency into image classification to identify what specific objects appear in a scene.

The Image Similarity feature (VNFeaturePrintObservation) is demonstrated with an interactive game where participants redraw an original sketch and the model ranks how close each reproduction is to the source by computing a floating-point distance between feature print vectors.

## Key Topics

**Image Saliency**
Two new request types produce a heatmap pixel buffer (CVPixelBuffer of floats 0–1):
- Attention-based: where human eyes are drawn (faces, contrast, motion cues, horizons)
- Objectness-based: foreground subjects (up to three bounding boxes)
Use cases: smart photo cropping/pan-and-scan, graphical masking effects, feeding crops into a classifier.

**Image Classification**
Apple ships its Photos search network as a public Vision API. Key characteristics:
- Multi-label (each class has an independent confidence; scores do not sum to 1)
- 1000+ visually identifiable categories arranged in a taxonomy
- Classes exclude occupations, proper nouns, abstract concepts, offensive content
- Results must be filtered with `hasMinimumRecall` or `hasMinimumPrecision` rather than a single threshold

**Image Similarity (Feature Print)**
`VNGenerateImageFeaturePrintRequest` extracts an upper-layer feature vector from the classification network. `VNFeaturePrintObservation.computeDistance(_:to:)` returns a float: smaller = more similar. Works on any natural image regardless of taxonomy constraints.

**Face Landmarks — Revision 3**
- 76-point constellation (up from 65)
- Per-point confidence via `precisionEstimatesPerPoint`
- Improved pupil detection accuracy

**Face Capture Quality**
`VNDetectFaceCaptureQualityRequest` scores a face image 0–1 for lighting, focus, and expression quality. Intended for ranking multiple captures of the same subject (e.g., burst photos); not for cross-subject comparison.

**New Detectors**
- `VNDetectHumanRectanglesRequest` **[NEW]** — detects human upper body (head + torso); returns `VNDetectedObjectObservation`
- `VNRecognizeAnimalsRequest` **[NEW]** — detects cats and dogs with label; returns `VNRecognizedObjectObservation`

**Object Tracker Revision 2**
ML-based tracking; better occlusion handling, improved bounding box expansion, lower power consumption.

**Vision + Core ML: Multi-Input/Multi-Output**
New `inputFeatureName` and `featureProvider` properties on `VNCoreMLModel` allow passing multiple inputs (including multiple images) to a Core ML model. New `featureName` property on `VNCoreMLFeatureValueObservation` lets you distinguish multiple outputs by name.

## APIs & Frameworks

**Vision**
- `VNGenerateAttentionBasedSaliencyImageRequest` **[NEW]** — attention heatmap
- `VNGenerateObjectnessBasedSaliencyImageRequest` **[NEW]** — objectness heatmap
- `VNSaliencyImageObservation` **[NEW]** — result type; `.pixelBuffer` (CVPixelBuffer of floats), `.salientObjects` ([VNRectangleObservation])
- `VNClassifyImageRequest` **[NEW]** — multi-label on-device image classification
- `VNClassificationObservation.hasMinimumRecall(_:forPrecision:)` **[NEW]** — filter by recall at minimum precision
- `VNClassificationObservation.hasMinimumPrecision(_:forRecall:)` **[NEW]** — filter by precision at minimum recall
- `VNClassifyImageRequest.knownClassifications(forRevision:)` **[NEW]** — query full taxonomy
- `VNGenerateImageFeaturePrintRequest` **[NEW]** — feature print extraction
- `VNFeaturePrintObservation` **[NEW]** — feature print result; `computeDistance(_:to:)` returns Float
- `VNDetectFaceLandmarksRequest` — updated to revision 3 with 76-point constellation
- `VNFaceLandmarksRegion2D.precisionEstimatesPerPoint` **[NEW]** — per-landmark confidence
- `VNFaceLandmarks2D` — `.constellation` now configurable (`.constellation65Points` or `.constellation76Points`)
- `VNDetectFaceCaptureQualityRequest` **[NEW]** — face capture quality scoring
- `VNFaceObservation.faceCaptureQuality` **[NEW]** — Float quality score for ranking captures
- `VNDetectHumanRectanglesRequest` **[NEW]** — human upper-body detector
- `VNRecognizeAnimalsRequest` **[NEW]** — cat/dog detector with label
- `VNRecognizedObjectObservation` — extended to carry animal labels
- `VNSequenceRequestHandler` — used for frame-by-frame tracking
- `VNTrackObjectRequest` — revision 2 ML-based tracker **[updated]**
- `VNCoreMLModel.inputFeatureName` **[NEW]** — specify which model input the handler image maps to
- `VNCoreMLModel.featureProvider` **[NEW]** — supply additional inputs (images or values)
- `VNCoreMLFeatureValueObservation.featureName` **[NEW]** — identify which output an observation corresponds to
- `VNImageRequestHandler` — single-image handler
- `VNSequenceRequestHandler` — multi-frame sequence handler

## Code Highlights

Attention-based saliency:

```swift
let handler = VNImageRequestHandler(cgImage: cgImage)
let request = VNGenerateAttentionBasedSaliencyImageRequest()
request.revision = VNGenerateAttentionBasedSaliencyImageRequestRevision1
try handler.perform([request])
if let observation = request.results?.first as? VNSaliencyImageObservation {
    let heatmap = observation.pixelBuffer   // CVPixelBuffer of Floats 0–1
    let boxes = observation.salientObjects  // [VNRectangleObservation]
}
```

Multi-label image classification with precision/recall filtering:

```swift
let request = VNClassifyImageRequest()
try handler.perform([request])
let observations = request.results as! [VNClassificationObservation]
// High-recall search: maximize retrieval, allow some false positives
let highRecall = observations.filter { $0.hasMinimumRecall(0.8, forPrecision: 0.5) }
// High-precision search: only confident predictions
let highPrecision = observations.filter { $0.hasMinimumPrecision(0.9, forRecall: 0.5) }
```

Feature print similarity:

```swift
let printRequest = VNGenerateImageFeaturePrintRequest()
try VNImageRequestHandler(cgImage: imageA).perform([printRequest])
let printA = printRequest.results!.first as! VNFeaturePrintObservation

try VNImageRequestHandler(cgImage: imageB).perform([printRequest])
let printB = printRequest.results!.first as! VNFeaturePrintObservation

var distance: Float = 0
try printA.computeDistance(&distance, to: printB)
// Smaller distance → more similar
```

## Takeaways
- Vision 2019 adds end-to-end image understanding pipelines: saliency → crop → classify, or saliency → feature print → similarity search, entirely on-device.
- Multi-label classification with `hasMinimumPrecision`/`hasMinimumRecall` replaces fixed-threshold filtering with a principled precision-recall tradeoff appropriate for each use case.
- Face Capture Quality is designed exclusively for ranking captures of the same person; do not use it to compare different subjects or set an absolute acceptance threshold.
- The new `VNFeaturePrintObservation` is taxonomy-agnostic: it can surface similarity for object categories the classification network was never explicitly trained to name.

---
_Source: WWDC19 Session 222 page (abstract, transcript, resource links, and sample code references)._
