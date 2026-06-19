# What's New in AVFoundation
**WWDC21 · Session 10146** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10146/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
AVFoundation gains three significant improvements in 2021. The headline change is a completely revamped async property-loading API for `AVAsset` and related classes that leverages Swift's new `async`/`await` concurrency model, replacing the verbose and error-prone `loadValuesAsynchronously(forKeys:)` pattern. The old synchronous properties are being deprecated for Swift clients in a future release. The second feature delivers per-frame timed metadata to custom video compositor callbacks, enabling GPS, telemetry, or any track-synchronized data to influence frame composition. Third, macOS gains authoring and ingestion support for iTunes Timed Text (`.itt`) and Scenarist Closed Captions (`.scc`) caption file formats.

## Key Topics

### AVAsset Async Property Inspection
The new `load(_:)` method accepts one or more typed `AVAsyncProperty` identifiers and returns their values asynchronously using `async`/`await`. Each property identifier carries its result type at compile time, making the API fully type-safe — no more string-keyed property names. Loading multiple properties in a single call allows AVFoundation to batch I/O efficiently. The companion `status(of:)` method returns a four-case enum (`notYetLoaded`, `loading`, `loaded(Value)`, `failed(Error)`) for checking property state without suspending. Calling `load(_:)` a second time on an already-loaded property returns the cached value immediately. The old `loadValuesAsynchronously(forKeys:)` + synchronous-property pattern is deprecated for Swift.

### Custom Video Composition with Timed Metadata
Custom video compositors (`AVVideoCompositing`) can now receive per-frame timed metadata from designated metadata tracks. Setup requires setting `sourceSampleDataTrackIDs` on `AVMutableVideoComposition` to enumerate all relevant metadata track IDs, and setting `requiredSourceSampleDataTrackIDs` on each `AVMutableVideoCompositionInstruction`. Inside the composition callback, `AVAsynchronousVideoCompositionRequest.sourceTimedMetadata(byTrackID:)` returns an `AVTimedMetadataGroup`, while `sourceSampleBuffer(byTrackID:)` returns raw `CMSampleBuffer` bytes for lower-level access.

### Caption File Authoring (macOS)
New macOS APIs enable authoring, ingesting, and previewing iTunes Timed Text (`.itt`) and Scenarist Closed Captions (`.scc`, CEA-608) files. `AVCaption` models a single caption with text, position, timing, and style. `AVAssetWriterInputCaptionAdaptor` writes captions to file. `AVCaptionConversionValidator` validates caption streams against format-specific constraints (e.g., bit budget per character for `.scc`) and suggests timestamp adjustments to achieve compliance. `AVAssetReaderOutputCaptionAdaptor` reads captions back. `AVCaptionRenderer` renders captions to a `CGContext` for preview.

## APIs & Frameworks

**AVFoundation — Async Asset Inspection**
- `AVAsset.load(_:) async throws -> Value` **[NEW]** — primary async property loader; single or multiple identifiers
- `AVAsset.status(of:) -> AVAsyncProperty.Status<Value>` **[NEW]**
- `AVAsyncProperty.Status` enum **[NEW]** — `.notYetLoaded`, `.loading`, `.loaded(Value)`, `.failed(Error)`
- `AVAsset.load(.duration) -> CMTime` **[NEW async variant]**
- `AVAsset.load(.tracks) -> [AVAssetTrack]` **[NEW async variant]**
- `AVAsset.load(.metadata) -> [AVMetadataItem]` **[NEW async variant]**
- `AVAsset.loadTrack(withTrackID:) async throws -> AVAssetTrack?` **[NEW]**
- `AVAsset.loadTracks(withMediaType:) async throws -> [AVAssetTrack]` **[NEW]**
- `AVAsset.loadTracks(withMediaCharacteristic:) async throws -> [AVAssetTrack]` **[NEW]**
- `AVAsset.loadMetadata(for:) async throws -> [AVMetadataItem]` **[NEW]**
- `AVAsset.loadChapterMetadataGroups(withTitleLocale:) async throws` **[NEW]**
- `AVAsset.loadChapterMetadataGroups(bestMatchingPreferredLanguages:) async throws` **[NEW]**
- `AVAsset.loadMediaSelectionGroup(for:) async throws -> AVMediaSelectionGroup?` **[NEW]**
- `AVAssetTrack.loadSegment(forTrackTime:) async throws` **[NEW]**
- `AVAssetTrack.loadSamplePresentationTime(forTrackTime:) async throws` **[NEW]**
- `AVAssetTrack.loadMetadata(for:) async throws -> [AVMetadataItem]` **[NEW]**
- `AVAssetTrack.loadAssociatedTracks(ofType:) async throws -> [AVAssetTrack]` **[NEW]**
- `AVMetadataItem.load(_:) async throws` **[NEW async variant]**
- `AVAsset.loadValuesAsynchronously(forKeys:completionHandler:)` — **[DEPRECATED for Swift]**

**AVFoundation — Custom Video Composition with Metadata**
- `AVMutableVideoComposition.sourceSampleDataTrackIDs: [CMPersistentTrackID]` **[NEW]**
- `AVMutableVideoCompositionInstruction.requiredSourceSampleDataTrackIDs: [CMPersistentTrackID]` **[NEW]**
- `AVAsynchronousVideoCompositionRequest.sourceSampleDataTrackIDs: [CMPersistentTrackID]` **[NEW]**
- `AVAsynchronousVideoCompositionRequest.sourceTimedMetadata(byTrackID:) -> AVTimedMetadataGroup?` **[NEW]**
- `AVAsynchronousVideoCompositionRequest.sourceSampleBuffer(byTrackID:) -> CMSampleBuffer?` **[NEW]**
- `AVAssetWriterInputMetadataAdaptor` — existing; used to write timed metadata tracks

**AVFoundation — Caption File Authoring (macOS)**
- `AVCaption` **[NEW]** — model object for a single caption (text, time range, position, styling)
- `AVAssetWriterInputCaptionAdaptor` **[NEW]** — writes `AVCaption` objects to `.itt` or `.scc` files
- `AVCaptionConversionValidator` **[NEW]** — validates caption stream against format constraints; suggests fixes
- `AVAssetReaderOutputCaptionAdaptor` **[NEW]** — reads `AVCaption` objects from caption files
- `AVCaptionRenderer` **[NEW]** — renders captions to `CGContext` for preview

## Code Highlights

Loading a single property async:
```swift
func inspectAsset() async throws {
    let asset = AVAsset(url: movieURL)
    let duration = try await asset.load(.duration)
    myFunction(thatUses: duration)
}
```

Loading multiple properties in one call:
```swift
let (duration, tracks) = try await asset.load(.duration, .tracks)
```

Checking property status:
```swift
switch asset.status(of: .duration) {
case .notYetLoaded: break
case .loading: break
case .loaded(let duration): use(duration)
case .failed(let error): handle(error)
}
```

Video composition callback with metadata:
```swift
func startRequest(_ request: AVAsynchronousVideoCompositionRequest) {
    for trackID in request.sourceSampleDataTrackIDs {
        let metadata = request.sourceTimedMetadata(byTrackID: trackID)
    }
    request.finish(withComposedVideoFrame: composedFrame)
}
```

## Takeaways
- The new `load(_:)` API with `async`/`await` replaces the old three-step `loadValuesAsynchronously` pattern; it is type-safe, more concise, and batches I/O when multiple properties are requested together.
- The old synchronous AVFoundation property APIs will be deprecated for Swift in a future release — start migrating now.
- Custom video compositors can now receive per-frame GPS, telemetry, or other timed metadata by setting `sourceSampleDataTrackIDs` on the composition and instruction objects.
- macOS gains full authoring, ingestion, validation, and preview capabilities for `.itt` and `.scc` caption file formats via new `AVCaption`-family classes.

---
_Source: WWDC21 Session 10146 page (abstract, chapter summaries, code samples, and resource links)._
