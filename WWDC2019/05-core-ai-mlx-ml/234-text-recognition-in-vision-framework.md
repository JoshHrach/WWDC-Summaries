# Text Recognition in Vision Framework
**WWDC19 · Session 234** · [Watch](https://developer.apple.com/videos/play/wwdc2019/234/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
Vision Framework gains a full on-device text recognition pipeline in iOS 13 via the new `VNRecognizeTextRequest`. This session covers the two recognition paths (fast vs. accurate), the `VNDocumentCameraViewController` companion for document scanning, performance tuning options (region of interest, minimum text height, CPU-only mode), language-based correction with custom lexicons, progress management, and best practices for processing results using geometry, candidate lists, `NSDataDetector`, and Core ML classifiers.

## Key Topics

### VNRecognizeTextRequest — The New API **[NEW]**
- A single `VNRecognizeTextRequest` replaces the previous multi-step pipeline (VNDetectTextRectanglesRequest → Core ML character classifier → heuristic string reconstruction).
- Returns `VNRecognizedTextObservation` results per line/string, each carrying multiple ranked candidates.
- Results include bounding boxes (normalized image coordinates) for both the whole observation and any substring within it.

### Fast vs. Accurate Recognition Levels
- `VNRequestTextRecognitionLevel.fast` — character-by-character recognition using a lightweight ML model; optimized for real-time camera feeds; lower memory; limited support for rotated/stylized text.
- `VNRequestTextRecognitionLevel.accurate` — state-of-the-art neural network that reads words and sentences holistically (similar to how humans read); better for stylized fonts, rotated/perspective text, and natural language; higher memory; best run asynchronously.
- Both paths support an optional **language correction phase** that eliminates common misreadings using on-device language models.

### Choosing a Recognition Level
| Scenario | Recommended Level |
|---|---|
| Real-time barcode/serial number scanning | Fast |
| Opportunistic photo capture | Accurate |
| Post-processing existing images | Accurate |
| Memory-constrained environment | Fast |
| Natural language documents | Accurate |

### Language Configuration **[NEW]**
- `VNRecognizeTextRequest.recognitionLanguages` — set the target language (e.g., `["en-US"]`) to enable language-based correction; supported languages vary by recognition level and API revision.
- `VNRecognizeTextRequest.usesLanguageCorrection` — enable/disable language correction; disable when reading codes, serial numbers, or other non-natural-language strings.
- `VNRecognizeTextRequest.customWords` — array of domain-specific vocabulary strings (medical terms, product codes, reference numbers) that complement the language model; corrects cases where generic models would misread known terms.

### Performance Tuning
- `VNRequest.regionOfInterest` — normalized rect restricting processing to a subregion of the image; reduces noise and improves speed.
- `VNRecognizeTextRequest.minimumTextHeight` — fraction of image height (0.0–1.0); text shorter than this threshold is ignored; also causes the framework to downsample the image, reducing memory and execution time.
- `VNRequest.usesCPUOnly` — run only on CPU, leaving GPU/Neural Engine available for higher-priority tasks (e.g., concurrent ARKit rendering). **[NEW]**
- `VNRequest.revision` — pin to a specific revision so future model updates don't unexpectedly change behavior. Recommended for production.

### Progress Management **[NEW]**
- `VNRequest.progressHandler` — closure called with a fractional progress value (0.0–1.0) as recognition proceeds; use to update a progress indicator.
- Cancellation: call `cancel()` on the request to stop in-flight recognition.

### VNDocumentCameraViewController (VisionKit) **[NEW]**
- A pre-built system UI for capturing flat documents: finds the document, applies perspective correction, and evenly illuminates the scan — all automatically.
- Ships in the new **VisionKit** framework.
- Used in Notes, Mail, Files, and Messages (iOS 13).
- Delegate returns an array of `VNDocumentCameraScan` pages; each page exposes a `UIImage`/`CGImage` ready to pipe directly into a `VNRecognizeTextRequest`.

### Processing Results — Best Practices
1. **Candidate list**: Each `VNRecognizedTextObservation` provides `topCandidates(_:)` returning multiple ranked strings. For indexing/search, index multiple candidates to improve recall at the cost of precision.
2. **Geometry**: Use bounding boxes to spatially relate results (e.g., map receipt item names to prices by column alignment).
3. **NSDataDetector**: Parse results for addresses, URLs, dates, and phone numbers without writing custom parsers.
4. **Domain parsers**: Apply known structural rules (e.g., American phone number format) to filter and validate candidates.
5. **Evidence accumulation over frames**: For live camera, accept a result only after it appears consistently across N consecutive frames (e.g., 10) to suppress frame-to-frame noise.
6. **Core ML classifier**: Classify document type before running recognition (e.g., receipt vs. business card) using a Create ML model to select the appropriate post-processing pipeline automatically.

## APIs & Frameworks

### Vision — Text Recognition **[NEW]**
- `VNRecognizeTextRequest` — OCR request **[NEW]**
  - `recognitionLevel: VNRequestTextRecognitionLevel` — `.fast` or `.accurate` **[NEW]**
  - `recognitionLanguages: [String]` — BCP-47 language tags **[NEW]**
  - `usesLanguageCorrection: Bool` **[NEW]**
  - `customWords: [String]` — domain vocabulary **[NEW]**
  - `minimumTextHeight: Float` — fraction of image height **[NEW]**
  - `revision: Int` — pin API version **[NEW]**
  - `progressHandler: VNRequestProgressHandler?` — fractional progress callback **[NEW]**
  - `regionOfInterest: CGRect` — subregion to process (inherited from VNRequest)
  - `usesCPUOnly: Bool` — CPU-only execution (inherited from VNRequest) **[NEW]**
- `VNRecognizedTextObservation` — result per text line **[NEW]**
  - `topCandidates(_ maxCandidateCount: Int) -> [VNRecognizedText]` **[NEW]**
  - `boundingBox: CGRect` — normalized bounding box
- `VNRecognizedText` — a single recognition candidate **[NEW]**
  - `string: String` — recognized text
  - `confidence: VNConfidence`
  - `boundingBox(for range: Range<String.Index>) -> VNRectangleObservation?` — bounding box for a substring **[NEW]**
- `VNRequestTextRecognitionLevel` — enum: `.fast`, `.accurate` **[NEW]**
- `VNImageRequestHandler` — existing; create with `CGImage`/`CVPixelBuffer`/`URL`

### VisionKit **[NEW Framework]**
- `VNDocumentCameraViewController` — system document scanning UI **[NEW]**
  - `delegate: VNDocumentCameraViewControllerDelegate?`
- `VNDocumentCameraViewControllerDelegate` **[NEW]**
  - `documentCameraViewController(_:didFinishWith scan:)` — called with captured pages
- `VNDocumentCameraScan` — scanned document **[NEW]**
  - `pageCount: Int`
  - `imageOfPage(at pageIndex: Int) -> UIImage`

### Foundation
- `NSDataDetector` — parse recognized strings for phone numbers, URLs, addresses, dates

## Code Highlights

Basic accurate text recognition:
```swift
import Vision

let requestHandler = VNImageRequestHandler(cgImage: cgImage, options: [:])
let request = VNRecognizeTextRequest { request, error in
    guard let observations = request.results as? [VNRecognizedTextObservation] else { return }
    for observation in observations {
        guard let topCandidate = observation.topCandidates(1).first else { continue }
        print(topCandidate.string)
        print(observation.boundingBox)  // normalized coordinates
    }
}
request.recognitionLevel = .accurate
request.revision = VNRecognizeTextRequestRevision1
request.usesLanguageCorrection = true
try requestHandler.perform([request])
```

Real-time fast scanning with region of interest:
```swift
let request = VNRecognizeTextRequest(completionHandler: handleResults)
request.recognitionLevel = .fast
request.usesLanguageCorrection = false
request.regionOfInterest = CGRect(x: 0.1, y: 0.3, width: 0.8, height: 0.4)

// In AVCaptureVideoDataOutputSampleBufferDelegate:
func captureOutput(_ output: AVCaptureOutput, didOutput sampleBuffer: CMSampleBuffer, ...) {
    guard let pixelBuffer = CMSampleBufferGetImageBuffer(sampleBuffer) else { return }
    let handler = VNImageRequestHandler(cvPixelBuffer: pixelBuffer, options: [:])
    try? handler.perform([request])
}
```

Custom vocabulary for domain-specific terms:
```swift
request.recognitionLanguages = ["en-US"]
request.usesLanguageCorrection = true
request.customWords = ["HI11", "SKU-4892", "ACME-Corp"]
```

Document Camera integration:
```swift
import VisionKit

let controller = VNDocumentCameraViewController()
controller.delegate = self
present(controller, animated: true)

func documentCameraViewController(_ controller: VNDocumentCameraViewController,
                                   didFinishWith scan: VNDocumentCameraScan) {
    for pageIndex in 0..<scan.pageCount {
        let image = scan.imageOfPage(at: pageIndex)
        recognizeText(in: image.cgImage!)
    }
    dismiss(animated: true)
}
```

Progress management with cancellation:
```swift
request.progressHandler = { request, progress, error in
    DispatchQueue.main.async {
        self.progressView.progress = Float(progress)
    }
}
// Cancel on user request:
request.cancel()
```

Evidence accumulation across camera frames:
```swift
var candidateCounts: [String: Int] = [:]
let threshold = 10

func handleResults(_ request: VNRequest, _ error: Error?) {
    guard let observations = request.results as? [VNRecognizedTextObservation] else { return }
    for obs in observations {
        guard let text = obs.topCandidates(1).first?.string,
              isPhoneNumber(text) else { continue }
        candidateCounts[text, default: 0] += 1
        if candidateCounts[text]! >= threshold {
            DispatchQueue.main.async { self.displayResult(text) }
        }
    }
}
```

## Takeaways
- `VNRecognizeTextRequest` is a complete on-device OCR pipeline — no custom Core ML model or string-reconstruction heuristics needed.
- Choose **fast** for real-time camera feeds and code scanning; choose **accurate** for document post-processing and natural language content.
- Disable `usesLanguageCorrection` when reading codes/serials; provide `customWords` to handle domain-specific vocabulary that generic models misread.
- `VNDocumentCameraViewController` (VisionKit) handles perspective correction and even lighting automatically — always prefer it over manual camera capture for document scanning.
- Post-process results with the candidate list (for recall), spatial geometry (for structure), `NSDataDetector` (for entity extraction), and an evidence-accumulation buffer (for live camera stability).

---
_Source: WWDC19 Session 234 page (abstract, full transcript, and resource links)._
