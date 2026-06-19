# Discover Swift Enhancements in the Vision Framework
**WWDC24 · Session 10163** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10163/)

_Platforms:_ iOS 18, iPadOS 18, macOS Sequoia 15, tvOS 18, visionOS 2, watchOS 11

## Overview
The Vision framework receives a comprehensive Swift-first redesign in iOS 18. The new API drops the `VN` prefix convention, replaces completion-handler-based requests with native `async`/`await`, and returns observations directly from `perform()` calls rather than through side-channel properties. The result is that barcode detection goes from ~10 lines to 3, and the new API is fully Swift 6 / strict-concurrency-safe.

The session also introduces two new capabilities: **Image Aesthetics Scores** (judging overall photo quality and identifying utility images) and **Holistic Body Pose** (detecting body and both hands in a single request). Vision will continue adding new features exclusively through the new Swift API going forward.

## Key Topics

### New Vision API Design
Every Vision request is now a value type named without the `VN` prefix (e.g., `DetectBarcodesRequest`). Requests are `async` and return typed observation arrays directly. A single `ImageRequestHandler` can still batch multiple requests together for efficiency. The new `performAll(_:)` variant returns results as an async stream so each request's observations are available as soon as that request finishes, without waiting for others.

### Swift Concurrency Optimization
`TaskGroup` is the recommended pattern for processing multiple images in parallel. The session recommends limiting concurrent Vision tasks to 5 to balance throughput and memory usage, as Vision requests can be memory-intensive.

### Updating Existing Apps
Migration is mechanical: remove the `VN` prefix, delete completion handlers, `await` the `perform()` call, and use observations returned directly. The `VNImageRequestHandler` pattern still works but is optional for single-request flows.

### Neural Engine Compute Devices
On devices with a Neural Engine, Vision may remove CPU/GPU support for some requests. Use `supportedComputeDevices()` to check which compute paths are available for a given request type.

### New Capabilities: Image Aesthetics and Holistic Body Pose
`CalculateImageAestheticsScoresRequest` produces an `ImageAestheticsScoresObservation` with an `overallScore` (-1 to 1) and `isUtility` flag. `DetectHumanBodyPoseRequest` now has a `detectsHands` property that, when `true`, causes `HumanBodyPoseObservation` to include `rightHandObservation` and `leftHandObservation`.

## APIs & Frameworks

**Vision (new Swift API — all names without `VN` prefix)**
- `DetectBarcodesRequest` **[NEW Swift API]**
  - `.symbologies: [BarcodeSymbology]`
  - `perform(on:) async throws -> [BarcodeObservation]`
- `BarcodeObservation` **[NEW Swift API]**
  - `.boundingBox: NormalizedRect`
  - `.payloadString: String?`
- `RecognizeTextRequest` **[NEW Swift API]**
- `RecognizedTextObservation` **[NEW Swift API]**
- `GenerateObjectnessBasedSaliencyImageRequest` **[NEW Swift API]**
- `SaliencyImageObservation` **[NEW Swift API]**
- `ImageRequestHandler` **[NEW Swift API]**
  - `perform(_:) async throws -> (ObsA, ObsB)` (parameter pack) **[NEW]**
  - `performAll(_:)` returning async stream **[NEW]**
- `NormalizedRect.toImageCoordinates(_:origin:)` **[NEW]**
- `CalculateImageAestheticsScoresRequest` **[NEW]**
- `ImageAestheticsScoresObservation` **[NEW]**
  - `.overallScore: Float` (range -1 to 1)
  - `.isUtility: Bool`
- `DetectHumanBodyPoseRequest` **[NEW Swift API, enhanced]**
  - `.detectsHands: Bool` **[NEW]**
- `HumanBodyPoseObservation` **[NEW Swift API]**
  - `.rightHandObservation: HumanHandPoseObservation?` **[NEW]**
  - `.leftHandObservation: HumanHandPoseObservation?` **[NEW]**
- `supportedComputeDevices()` on request types **[NEW]**
- All 31 request types available in new Swift naming

## Code Highlights

**3-line barcode detection (new API):**
```swift
let request = DetectBarcodesRequest()
let barcodeObservations = try await request.perform(on: image)
```

**Coordinate conversion:**
```swift
let imageRect = observation.boundingBox.toImageCoordinates(imageSize, origin: .upperLeft)
```

**Parallel image processing with TaskGroup:**
```swift
await withTaskGroup(of: CGImage?.self) { group in
    for image in images.prefix(5) {
        group.addTask { try? await generateThumbnail(for: image) }
    }
}
```

**Holistic body pose:**
```swift
var request = DetectHumanBodyPoseRequest()
request.detectsHands = true
let observations = try await request.perform(on: image)
let rightHand = observations.first?.rightHandObservation
```

## Takeaways
- Migrate to the new `VN`-prefix-free Swift API to get `async`/`await` ergonomics and Swift 6 safety — migration is mostly a find/replace of type names.
- Use `ImageRequestHandler.performAll(_:)` when you need first-available results from multiple concurrent requests.
- Adopt `CalculateImageAestheticsScoresRequest` to surface high-quality or utility photos in photo-selection experiences.
- Enable `detectsHands = true` on `DetectHumanBodyPoseRequest` to get a unified body + hands skeleton in one call.

---
_Source: WWDC24 Session 10163 page (abstract, chapter summaries, code samples, and resource links)._
