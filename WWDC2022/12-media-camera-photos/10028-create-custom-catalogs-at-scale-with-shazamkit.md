# Create Custom Catalogs at Scale with ShazamKit
**WWDC22 · Session 10028** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10028/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, watchOS 9, tvOS 16

## Overview
This session presents major improvements to the ShazamKit custom catalog workflow, focusing on three new capabilities: a command-line tool (Shazam CLI) for batch catalog creation, a new `signatureFromAsset` API that eliminates manual audio buffer management, and the Timed Media Item API for precise time-range-based content synchronization within a matched audio source.

The session revisits the FoodMath app demo from WWDC21 and shows how a math quiz can be precisely synced to specific moments in a video using time ranges attached to `SHMediaItem` objects. It also covers best practices for structuring custom catalogs across large content libraries and introduces frequency skew as a technique to differentiate repeated audio sources (e.g., the same intro music used across multiple podcast episodes).

## Key Topics

### Shazam CLI (macOS 13)
A new command-line tool ships with macOS Ventura that automates catalog creation:
- `shazam signature` — converts any media file with an audio track into a `.shazamsignature`
- `shazam custom-catalog create` — combines a signature file and a CSV of media item metadata into a `.shazamcatalog`
- `shazam custom-catalog update` — adds new content to an existing catalog
- `shazam custom-catalog display` — inspects catalog contents
- Supports add, remove, and export of individual signatures and media items
- CSV headers map to `SHMediaItem` property keys (see `--help` for mapping)
- Scriptable for batch processing large content libraries

### signatureFromAsset
`SHSignatureGenerator` gains a new static method `signatureFromAsset(_:)` that accepts any `AVAsset` with an audio track. Multiple audio tracks are mixed together automatically. No manual buffer extraction or sample rate management required.

### Timed Media Items
`SHMediaItem` now supports a `timeRanges` property — an array of `Range<TimeInterval>` values — that restricts when the media item is returned in match callbacks. ShazamKit delivers match callbacks at the start and end of each time range. Only media items whose time ranges overlap the current match offset are returned; items with no time ranges are always returned last (good for global episode-level metadata). If all media items have time ranges and none are in scope, a synthetic media item with basic match properties (`predictedCurrentMatchOffset`, `frequencySkew`) is always returned.

### AsyncSequence-Based Session Results
`SHSession` now exposes results as an `AsyncSequence` yielding an enum of `.match`, `.noMatch`, and `.error` cases, enabling clean `for await` loops without delegate boilerplate.

### Catalog Composition
Multiple catalog files can be merged at runtime using `SHCustomCatalog.add(from:)`, allowing apps to load per-episode or per-track catalogs on demand and combine them into a single working catalog.

### Frequency Skew Differentiation
Audio played back with frequencies shifted by 1–5% is still recognizable to ShazamKit but sounds identical to humans. `SHMatchedMediaItem.frequencySkew` reports the detected skew as a `Float` (0 = unskewed, 0.01 = 1%). `SHMediaItem.frequencySkewRanges` — an array of `Range<Float>` — restricts a media item to matches within a specific skew band, enabling disambiguation of repeated audio (e.g., the same intro jingle used across seasons).

## APIs & Frameworks

### ShazamKit
- `SHSignatureGenerator.signatureFromAsset(_:) async throws -> SHSignature` **[NEW]** — creates a signature from any `AVAsset`; mixes multiple audio tracks
- `SHMediaItem.timeRanges: [Range<TimeInterval>]` **[NEW]** — restricts the media item to specific time windows within the reference signature; set via `.timeRanges` initializer property key
- `SHMediaItem.frequencySkewRanges: [Range<Float>]` **[NEW]** — restricts media item to matches with a specific frequency skew band; set via `.frequencySkewRanges` initializer property key
- `SHMatchedMediaItem.frequencySkew: Float` — detected frequency skew of the matched audio
- `SHMatchedMediaItem.predictedCurrentMatchOffset: TimeInterval` — estimated current playback position within the matched reference
- `SHSession.results: AsyncSequence` **[NEW]** — async sequence of `SHSession.Result` (`.match(SHMatch)`, `.noMatch`, `.error(Error)`)
- `SHCustomCatalog.add(from: URL) throws` — merges an existing catalog file into the current catalog
- `SHMediaItemProperty.timeRanges` **[NEW]** — dictionary key for time ranges in `SHMediaItem(properties:)`
- `SHMediaItemProperty.frequencySkewRanges` **[NEW]** — dictionary key for frequency skew ranges
- `SHRange` **[NEW]** — Objective-C counterpart to Swift `Range<Float>` for frequency skew ranges

## Code Highlights

Async matching loop using the new `AsyncSequence` API:
```swift
for await case .match(let match) in session.results {
    self.matchResult = match.mediaItems.reduce(into: MatchResult()) { result, item in
        result.title = result.title ?? item.title
        result.equation = result.equation ?? item.equation
    }
}
```

Creating a timed media item:
```swift
let mediaItem = SHMediaItem(properties: [
    .title: "Question 1",
    .timeRanges: [26.0..<35.0]
])
```

Frequency skew restriction:
```swift
let mediaItem = SHMediaItem(properties: [
    .title: "Season 2 Episode 1",
    .frequencySkewRanges: [0.03..<0.04]  // 3–4% skew
])
```

Loading multiple catalogs at runtime:
```swift
let catalog = SHCustomCatalog()
try catalog.add(from: episode1URL)
try catalog.add(from: episode2URL)
```

## Takeaways
- The Shazam CLI (ships with macOS Ventura) automates catalog creation from CSV metadata and media files, making large-scale catalog management scriptable.
- Create one signature per media asset — longer signatures improve match accuracy and avoid query-signature overlap issues at reference boundaries.
- Use `timeRanges` on `SHMediaItem` to drive timed UI experiences with zero custom timing logic in the app.
- Frequency skew (1–5%) is a practical technique to differentiate repeated audio content (intros, jingles) across multiple catalog entries without audible distortion.

---
_Source: WWDC22 Session 10028 page (abstract, chapter summaries, code samples, and resource links)._
