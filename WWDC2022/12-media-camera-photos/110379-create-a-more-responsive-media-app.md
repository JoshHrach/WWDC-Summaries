# Create a More Responsive Media App
**WWDC22 · Session 110379** · [Watch](https://developer.apple.com/videos/play/wwdc2022/110379/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
This session focuses on keeping media apps responsive by replacing synchronous AVFoundation I/O operations with new async/await APIs introduced in iOS 16 / macOS Ventura. The presenter covers three areas: new async image generation methods on `AVAssetImageGenerator`, new async composition and video composition constructors, and improvements to `AVAssetResourceLoader` for locally available media.

A major theme is the deprecation of `loadValuesAsynchronously(forKeys:)` and direct synchronous property access on `AVAsset` in Swift, in favor of the type-safe `async load(_:)` method introduced in 2021. The session explains why string-based key loading is error-prone and demonstrates how compile-time safety prevents accidental I/O on the main thread.

The third topic introduces `AVAssetResourceLoader.entireLengthAvailableOnDemand`, a new flag for apps that provide custom data loading from local storage, which allows the asset to skip unnecessary buffering and start playback faster.

## Key Topics

### Async Image Generation
`AVAssetImageGenerator` gains two new async methods. The single-image `image(at:)` method returns a tuple of the image and its actual time, freeing the calling thread while frame data loads. The multi-image `images(for:)` method returns an `AsyncSequence` of results — each element is either a `.success` with the requested time, image, and actual time, or a `.failed` with an error. Tolerance parameters (`requestedTimeToleranceBefore`/`After`) should be set as wide as acceptable to let the generator pick the nearest I-frame and minimize data loading.

### Async Composition and Video Composition
`AVMutableComposition.insertTimeRange(_:of:at:)` now has an async counterpart that loads the asset's tracks asynchronously, eliminating the need to pre-load them. `AVVideoComposition` gains an async `videoComposition(withPropertiesOf:)` factory and `isValid(for:timeRange:validationDelegate:)` method. Since `AVMutableComposition` is backed by in-memory structures, its synchronous property accessors remain available.

### Deprecation of Legacy Async KVO Loading
`loadValuesAsynchronously(forKeys:)` and direct synchronous properties on `AVAsset`, `AVAssetTrack`, and `AVMetadataItem` are deprecated in Swift in favor of `async load(_:)`. The old API used string keys prone to typos; the new API uses type-safe identifiers checked at compile time.

### AVAssetResourceLoader — Local Media Optimization
For apps storing media bytes in custom formats and using `AVAssetResourceLoader` to serve them, the new `entireLengthAvailableOnDemand` flag tells the asset that data is available synchronously. This reduces memory usage (no caching buffer) and reduces playback start latency. It must not be used for any network-backed storage.

## APIs & Frameworks

### AVFoundation — AVAssetImageGenerator
- `AVAssetImageGenerator.image(at:) async throws -> (image: CGImage, actualTime: CMTime)` **[NEW]** — single async image at a given time
- `AVAssetImageGenerator.images(for:) -> AsyncSequence` **[NEW]** — multi-image async sequence; takes `[CMTime]`
- `AVAssetImageGeneratorResult` — enum with `.success(requestedTime:image:actualTime:)` and `.failed(requestedTime:error:)` cases
- `requestedTimeToleranceBefore: CMTime` — tolerance before requested time for I-frame selection
- `requestedTimeToleranceAfter: CMTime` — tolerance after requested time for I-frame selection
- `copyCGImage(at:actualTime:)` — deprecated synchronous path; avoid on main thread

### AVFoundation — AVMutableComposition
- `AVMutableComposition.insertTimeRange(_:of:at:) async throws` **[NEW]** — async version that loads asset tracks as needed
- `AVMutableComposition.addMutableTrack(withMediaType:preferredTrackID:)` — existing synchronous method (safe; in-memory)
- `AVMutableCompositionTrack.insertTimeRange(_:of:at:)` — synchronous insertion of track segments (safe; in-memory)

### AVFoundation — AVVideoComposition
- `AVVideoComposition.videoComposition(withPropertiesOf:) async throws` **[NEW]** — async factory that loads tracks and duration
- `AVMutableVideoComposition.videoComposition(withPropertiesOf:) async throws` **[NEW]** — async mutable version
- `AVVideoComposition.isValid(for:timeRange:validationDelegate:) async throws -> Bool` **[NEW]** — async validation

### AVFoundation — Asset Inspection
- `AVAsset.load(_:) async throws` — type-safe async property loading (introduced WWDC21)
- `AVAsset.loadValuesAsynchronously(forKeys:completionHandler:)` — **deprecated in Swift**
- `AVAsset.statusOfValue(forKey:error:)` — **deprecated in Swift**
- Affected classes: `AVAsset`, `AVAssetTrack`, `AVMetadataItem`, and subclasses

### AVFoundation — AVAssetResourceLoader
- `AVAssetResourceLoader.entireLengthAvailableOnDemand: Bool` **[NEW]** — hint that data is available on-demand without caching; local media only
- `AVAssetResourceLoaderDelegate` — existing protocol for custom data serving

## Code Highlights

Async single thumbnail with wide tolerances:
```swift
func thumbnail() async throws -> UIImage {
    let generator = AVAssetImageGenerator(asset: asset)
    generator.requestedTimeToleranceBefore = .zero
    generator.requestedTimeToleranceAfter = CMTime(seconds: 3, preferredTimescale: 600)
    let thumbnail = try await generator.image(at: time).image
    return UIImage(cgImage: thumbnail)
}
```

Async timeline thumbnails using `AsyncSequence`:
```swift
func timelineThumbnails(for times: [CMTime]) async {
    for await result in generator.images(for: times) {
        updateThumbnail(for: result.requestedTime, with: (try? result.image) ?? placeholder)
    }
}
```

Async composition insertion:
```swift
let composition = AVMutableComposition()
try await composition.insertTimeRange(timeRange, of: asset, at: startTime)
```

Enabling local-only resource loader optimization:
```swift
resourceLoader.entireLengthAvailableOnDemand = true
```

## Takeaways
- Replace `copyCGImage(at:)` on the main thread with the new `image(at:) async` method; use `images(for:) async` for timeline scrubber thumbnails.
- Use wide tolerances with `AVAssetImageGenerator` to let it pick nearby I-frames and avoid loading dependent frames unnecessarily.
- `loadValuesAsynchronously(forKeys:)` is deprecated in Swift; migrate to `async load(_:)` for compile-time safety and clearer code.
- Set `entireLengthAvailableOnDemand = true` on `AVAssetResourceLoader` when your media bytes are stored locally to reduce buffering overhead and improve playback start time.

---
_Source: WWDC22 Session 110379 page (abstract, chapter summaries, code samples, and resource links)._
