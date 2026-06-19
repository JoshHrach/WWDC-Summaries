# Read Documents Using the Vision Framework
**WWDC25 · Session 272** · [Watch](https://developer.apple.com/videos/play/wwdc2025/272/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, tvOS 26, visionOS 26

## Overview
The Vision framework introduces two major new APIs in this session: `RecognizeDocumentsRequest` for structured document understanding and `DetectLensSmudgeRequest` for identifying photos taken with a smudged camera lens. All Vision APIs run entirely on-device, and the 31-API framework is now growing to 33.

`RecognizeDocumentsRequest` goes beyond the existing `RecognizeTextRequest` by extracting the structure of a document — tables with rows and columns, paragraphs, lists, barcodes, and detected data (emails, phone numbers, URLs, dates, tracking numbers, flight numbers, and more). The session demonstrates an iPad app that scans a customer sign-up sheet and automatically parses rows into contacts.

The session also covers a modernized hand pose detection model that is smaller, faster, and more accurate, though the joint positions differ from the old model, requiring retraining of ML classifiers.

## Key Topics

### Reading Documents (RecognizeDocumentsRequest)
The new request returns a `DocumentObservation` with a hierarchical structure: document → containers (text, tables, lists, barcodes) → cells / items. Tables expose a 2D array of cells accessible by rows or columns, with each cell having a `rowRange` and `columnRange` for spanning cells. Text containers offer `transcript`, `lines`, `paragraphs`, `words`, and `detectedData`. Detected data uses the new `DataDetection` framework to identify emails, phone numbers, postal addresses, URLs, dates, measurements, monetary amounts, tracking numbers, payment identifiers, and flight numbers. Document recognition supports 26 languages.

### Camera Lens Smudge Detection (DetectLensSmudgeRequest)
Returns a `LensSmudgeObservation` with a `confidence` score from 0 to 1. Developers choose a threshold (e.g., 0.9) to filter poor quality images. Caveats: motion blur, long exposure, or images of clouds/fog can produce false positives. Complementary APIs include `DetectFaceCaptureQualityRequest` (face images) and `CalculateImageAestheticScoresRequest` (general images).

### Hand Pose Update
`DetectHandPoseRequest` is backed by a new smaller, modernized model. Still detects 21 joints, but with improved accuracy, less memory, and lower latency. Existing ML hand pose and action classifiers trained on the old model should be retrained.

## APIs & Frameworks

**Vision**
- **`RecognizeDocumentsRequest`** **[NEW]** — structured document recognition
- `DocumentObservation` **[NEW]** — hierarchical document result
  - `.document` — top-level container
  - `DocumentObservation.Container.Table` **[NEW]** — 2D cell grid
  - `.tables`, `.lists`, `.text`, `.barcodes` — container properties
  - `Table.rows`, `Table.columns`, `Table.boundingRegion`
  - `TableCell.rowRange`, `TableCell.columnRange`, `TableCell.content`
  - `Container.text.transcript`, `.lines`, `.paragraphs`, `.words`, `.detectedData`
- **`DetectLensSmudgeRequest`** **[NEW]** (also referred to as `DetectCameraLensSmudgeRequest`)
- `LensSmudgeObservation.confidence` **[NEW]**
- `DetectHandPoseRequest` — updated underlying model **[CHANGED]**
- `HandPoseObservation` — 21 joints, unchanged interface
- `RecognizeTextRequest` — existing line-level text recognition
- `DetectFaceCaptureQualityRequest` — capture quality scoring
- `CalculateImageAestheticScoresRequest` — aesthetic/utility scoring

**DataDetection** (new framework used internally)
- Detected data types: `.emailAddress`, `.phoneNumber`, `.postalAddress`, `.link`, `.calendarEvent`, `.measurement`, `.money`, `.trackingNumber`, `.paymentIdentifier`, `.flightNumber`

## Code Highlights

```swift
// Detect a table and parse contacts
let request = RecognizeDocumentsRequest()
let observations = try await request.perform(on: imageData)
let table = observations.first?.document.tables.first

for row in table.rows {
    let name = row.first?.content.text.transcript
    for cell in row.dropFirst() {
        for data in cell.content.text.detectedData {
            switch data.match.details {
            case .emailAddress(let email): ...
            case .phoneNumber(let phone): ...
            default: break
            }
        }
    }
}
```

```swift
// Lens smudge detection
let request = DetectLensSmudgeRequest()
let observation = try await request.perform(on: imageData)
if observation.confidence > 0.9 { /* reject image */ }
```

## Takeaways
- Adopt `RecognizeDocumentsRequest` to replace complex manual table-parsing code that previously required spatial geometry calculations.
- Use `DetectLensSmudgeRequest` before processing user-captured images to improve pipeline quality.
- Retrain any existing ML hand pose or action classifiers to take advantage of the improved model accuracy.
- Pair smudge detection with `DetectFaceCaptureQualityRequest` or `CalculateImageAestheticScoresRequest` for comprehensive image quality gating.

---
_Source: WWDC25 Session 272 page (abstract, chapter summaries, code samples, and resource links)._
