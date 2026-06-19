# Extract Document Data Using Vision
**WWDC21 · Session 10041** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10041/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session covers Vision framework enhancements for document analysis: a new revision of barcode detection (Revision 2) supporting additional symbologies and corrected region-of-interest behavior, expanded language support in text recognition, and a brand-new `VNDetectDocumentSegmentationRequest` that uses machine learning to segment and extract a document from an image with real-time performance on the Neural Engine.

A practical demo ties all three together — document segmentation, barcode reading, rectangle detection, OCR, and a Core ML classifier — to scan survey cards from camera images.

## Key Topics

**Barcode Detection Revision 2**
`VNDetectBarcodesRequestRevision2` adds support for Codabar, GS1DataBar (including Expanded and Limited), MicroPDF, and MicroQR symbologies. Critically, it fixes bounding box reporting so that results are returned relative to the specified Region of Interest (ROI), consistent with all other Vision requests. (Revision 1 always reported relative to the full image.) Passing an empty symbologies array detects all supported types. For efficiency, specify only the symbologies relevant to your use case.

**Text Recognition Language Expansion**
`VNRecognizeTextRequest` in iOS 15 significantly expands supported languages (query dynamically via `supportedRecognitionLanguages()`). In the Fast path, additional Latin character sets (e.g., German umlauts) are supported. In the Accurate path, non-Latin languages like Chinese require selecting that language as the primary/first entry — this controls which recognition model is loaded. Language order matters for ambiguity resolution.

**Document Segmentation Detection (NEW)**
`VNDetectDocumentSegmentationRequest` is a new ML-based request trained on sheets of paper, signs, notes, receipts, labels, and more. It returns:
- A low-resolution segmentation mask (pixel-level confidence of belonging to the document)
- Four corner points of the detected quadrilateral

Runs in real time on the Neural Engine (used internally by `VNDocumentCamera` in VisionKit on modern devices). Compared to `VNDetectRectanglesRequest`: the document detector works on non-rectangular documents, provides a segmentation mask, finds only one document, and requires a Neural Engine for real-time use. `VNDetectRectanglesRequest` is a traditional CV algorithm that runs on CPU, finds multiple rectangles (potentially nested), but is less robust to folds/obscured corners.

**Combining Vision with Core ML**
The demo shows using `VNCoreMLRequest` with a Core ML image classifier trained in Create ML to analyze checkbox images extracted via perspective correction, integrating Vision, Core Image, and Core ML into a complete pipeline.

## APIs & Frameworks

- **Vision** framework
- `VNDetectBarcodesRequest` — barcode detection
  - `VNDetectBarcodesRequestRevision2` **[NEW]** — new revision
  - New symbologies **[NEW]**: `.codabar`, `.gs1DataBar`, `.gs1DataBarExpanded`, `.gs1DataBarLimited`, `.microPDF417`, `.microQR`
  - `symbologies: [VNBarcodeSymbology]` — empty array = all symbologies
  - `VNBarcodeObservation.payloadStringValue` — decoded barcode content
  - ROI-relative bounding box behavior fixed in Revision 2
- `VNRecognizeTextRequest` — OCR / text recognition
  - `supportedRecognitionLanguages()` **[UPDATED]** — returns expanded language list
  - `recognitionLanguages: [String]` — first language determines model in Accurate path
  - Fast path: extended Latin support **[NEW]**
  - Accurate path: Chinese, and additional non-Latin models **[NEW]**
  - `recognitionLevel`: `.fast` / `.accurate`
  - Returns `[VNRecognizedTextObservation]`
  - `VNRecognizedTextObservation.topCandidates(_:)` → `[VNRecognizedText]`
- `VNDetectDocumentSegmentationRequest` **[NEW]** — ML-based document detection
  - Returns `[VNDocumentObservation]` (subclass of `VNRectangleObservation`)
  - Properties: `pixelBuffer` (segmentation mask), `topLeft/topRight/bottomLeft/bottomRight` corner points
  - Real-time on Neural Engine; also runs on GPU/CPU (not real-time)
  - Used internally by `VNDocumentCamera` (VisionKit) on modern devices
- `VNDetectRectanglesRequest` — traditional rectangle detector
  - `minimumSize: Float` — minimum rectangle size as fraction of image (default 0.2)
  - `maximumObservations: Int` — 0 = return all; default = 1 (most prominent)
  - Returns `[VNRectangleObservation]`
- `VNImageRequestHandler` — single-image request executor
- `VNCoreMLRequest` — runs Core ML model as Vision request
- `VNCoreMLModel` — wraps `MLModel` for Vision
- `VNClassificationObservation` — result of image classifier
- `VNRectangleObservation` — corner points: `topLeft`, `topRight`, `bottomLeft`, `bottomRight`
- **VisionKit** — `VNDocumentCamera` now uses `VNDetectDocumentSegmentationRequest` on Neural Engine devices
- **Core Image** — `CIFilter("CIPerspectiveCorrection")` — perspective-corrects extracted document regions
- **Core ML** / **Create ML** — image classifier for checkbox detection (`MLModel`, `VNCoreMLModel`, `VNCoreMLRequest`)

## Code Highlights

Barcode detection with Revision 2 and QR + EAN8:
```swift
let detectBarcodesRequest = VNDetectBarcodesRequest()
detectBarcodesRequest.revision = VNDetectBarcodesRequestRevision2
detectBarcodesRequest.symbologies = [.qr, .ean8]
try imageRequestHandler.perform([detectBarcodesRequest])
let barcodes = detectBarcodesRequest.results as? [VNBarcodeObservation]
```

Document segmentation + perspective correction:
```swift
let documentDetectionRequest = VNDetectDocumentSegmentationRequest()
try requestHandler.perform([documentDetectionRequest])
guard let document = documentDetectionRequest.results?.first,
      let documentImage = perspectiveCorrectedImage(from: inputImage, rectangleObservation: document)
else { return }
```

Rectangle detection with adjusted parameters:
```swift
let rectanglesDetection = VNDetectRectanglesRequest { request, _ in
    let rects = request.results as! [VNRectangleObservation]
    // sorted by vertical position
}
rectanglesDetection.minimumSize = 0.1
rectanglesDetection.maximumObservations = 0  // return all
```

Core ML checkbox classifier:
```swift
let model = try VNCoreMLModel(for: MLModel(contentsOf: modelURL))
let classificationRequest = VNCoreMLRequest(model: model)
try checkBoxRequestHandler.perform([classificationRequest])
let topClassification = (classificationRequest.results as? [VNClassificationObservation])?.first
```

## Takeaways

- `VNDetectDocumentSegmentationRequest` is a major new addition for document scanning — it is more robust than `VNDetectRectanglesRequest` for real-world documents and provides both a segmentation mask and corner points.
- `VNDetectBarcodesRequestRevision2` corrects long-standing ROI bounding box behavior and adds five new symbologies, making Vision more useful for healthcare and logistics use cases.
- For non-Latin text recognition (Chinese, etc.), always set the target language as the first element of `recognitionLanguages` in the Accurate path — it determines which model is loaded.
- Vision, Core Image, and Core ML compose naturally: use document segmentation + perspective correction to extract clean document images, then run OCR, barcode detection, and custom ML classifiers on the extracted content.

---
_Source: WWDC21 Session 10041 page (abstract, chapter summaries, code samples, and resource links)._
