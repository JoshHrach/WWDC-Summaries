# Create Custom Audio Experiences with ShazamKit
**WWDC21 · Session 10045** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10045/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15, watchOS 8

## Overview
ShazamKit is a new framework (iOS 15 / macOS 12) that exposes Shazam's exact audio matching technology to developers for matching against both the Shazam music catalog and custom developer-created catalogs. This code-along session focuses exclusively on custom catalog recognition: generating signatures from any audio source, associating metadata with those signatures, and using `AVAudioEngine` microphone input to match a live audio stream against the custom catalog.

The use case is a companion iOS app for an educational TV video — the app synchronizes interactive math questions to the correct moment in a video playing on another screen, using ShazamKit custom catalog recognition as the synchronization mechanism. No network calls or shared clocks are required; the audio itself serves as the sync signal.

## Key Topics

### Custom Catalog Concepts
A **custom catalog** (`SHCustomCatalog`) stores one or more **reference signatures** (`SHSignature`), each associated with a **media item** (`SHMediaItem`) containing custom metadata. At match time, ShazamKit returns a `SHMatchedMediaItem` — a subclass of `SHMediaItem` with the additional `predictedCurrentMatchOffset` property that reports the current position within the reference audio in seconds.

### Generating Reference Signatures
Use `SHSignatureGenerator` to convert audio buffers into a signature. Feed it `AVAudioPCMBuffer` objects (44100 Hz or other supported PCM rates) via `append(buffer:at:)`. Call `signature()` to produce the `SHSignature`. The finished signature can be saved as a `.shazamsignature` file for distribution.

Alternatively, pre-generated `.shazamsignature` files (provided with the session's sample project) can be loaded directly with `SHSignature(dataRepresentation:)`.

### Building the Catalog
1. Load the reference signature from a `.shazamsignature` file.
2. Create a `SHMediaItem` with a `properties` dictionary — use both predefined keys (`.title`, `.subtitle`, `.artist`) and custom extension keys.
3. Create `SHCustomCatalog()`.
4. Call `catalog.addReferenceSignature(signature, representing: [mediaItem])`.
5. Optionally save the catalog to disk as a `.shazamcatalog` file for cross-device sharing; load later with `SHCustomCatalog(from:)`.

### Streaming Microphone Matching
1. Create an `SHSession(catalog: customCatalog)` and set `session.delegate`.
2. Use `AVAudioEngine` — call `audioEngine.inputNode.installTap(onBus:bufferSize:format:block:)` to receive `AVAudioPCMBuffer` + `AVAudioTime` on each audio capture block.
3. In the tap block, call `session.matchStreamingBuffer(buffer, at: audioTime)`. ShazamKit generates and auto-updates the query signature internally; the `audioTime` ensures buffer continuity validation.
4. Implement `SHSessionDelegate` — `session(_:didFind:)` fires with an `SHMatch` containing an array of `SHMatchedMediaItem`.

### Using predictedCurrentMatchOffset
`SHMatchedMediaItem.predictedCurrentMatchOffset` is a `TimeInterval` that auto-updates to reflect the current playback position in the reference audio. Use it to find the appropriate content section by scanning a list of timed content items and returning the last one whose `offset` is ≤ the predicted offset.

`session(_:didFind:)` can fire multiple times for the same match (as offset updates). Deduplicate by comparing the new resolved content item to the previously displayed one; update the UI only on change.

## APIs & Frameworks

**ShazamKit** (`import ShazamKit`) — **[NEW iOS 15 / macOS 12]**

- `SHSession` **[NEW]** — audio matching session
  - `init(catalog: SHCatalog)` — create session against a catalog
  - `matchStreamingBuffer(_ buffer: AVAudioPCMBuffer, at time: AVAudioTime?)` **[NEW]** — feed mic audio; generates + maintains query signature internally
  - `delegate: SHSessionDelegate?`
- `SHSessionDelegate` **[NEW]**
  - `session(_:didFind:)` — match found; called repeatedly as offset updates
  - `session(_:didNotFindMatchFor:error:)` — no match
- `SHCustomCatalog : SHCatalog` **[NEW]** — developer-created catalog
  - `init()` — create empty catalog
  - `addReferenceSignature(_:representing:)` — add signature + associated media items
  - `add(from:)` — load from `.shazamcatalog` file URL (merge into existing catalog)
  - `write(to:)` — save catalog to disk as `.shazamcatalog`
  - `SHCatalog` (abstract base) — also used by `SHMusicCatalog` for Shazam music database
- `SHSignature` **[NEW]** — audio fingerprint
  - `init(dataRepresentation:)` — load from `.shazamsignature` file data
  - `dataRepresentation: Data` — serialize to save as `.shazamsignature`
- `SHSignatureGenerator` **[NEW]** — builds a signature from audio buffers
  - `append(_ buffer: AVAudioPCMBuffer, at time: AVAudioTime?)` — feed audio
  - `signature() throws -> SHSignature` — finalize and produce signature
- `SHMediaItem` **[NEW]** — metadata container
  - `init(properties: [SHMediaItemProperty: Any])` — create with property dictionary
  - Predefined keys: `.title`, `.subtitle`, `.artist`, `.genres`, `.artworkURL`, etc.
  - Custom keys: extend `SHMediaItemProperty` with `static let myKey = SHMediaItemProperty("myKey")`
  - `subscript(_ property: SHMediaItemProperty) -> Any?` — read property value
- `SHMatchedMediaItem : SHMediaItem` **[NEW]** — result of a match
  - `predictedCurrentMatchOffset: TimeInterval` **[NEW]** — auto-updating position in the reference audio (seconds)
  - `frequencySkew: Float` — playback speed difference between query and reference
- `SHMatch` **[NEW]** — contains array of `SHMatchedMediaItem`
  - `mediaItems: [SHMatchedMediaItem]`

**AVFAudio** (`import AVFAudio`)
- `AVAudioEngine` — audio processing graph
- `AVAudioInputNode.installTap(onBus:bufferSize:format:block:)` — capture microphone audio
- `AVAudioPCMBuffer` — PCM audio buffer; supports 44100 Hz and other ShazamKit-supported rates
- `AVAudioTime` — capture timestamp; pass to `matchStreamingBuffer` for continuity validation

## Code Highlights

Building a custom catalog:
```swift
import ShazamKit

// Load pre-generated reference signature from file
let signatureURL = Bundle.main.url(forResource: "FoodMath", withExtension: "shazamsignature")!
let signatureData = try Data(contentsOf: signatureURL)
let signature = try SHSignature(dataRepresentation: signatureData)

// Create media item with predefined and custom properties
extension SHMediaItemProperty {
    static let teacher = SHMediaItemProperty("teacher")
    static let episode = SHMediaItemProperty("episode")
}

let mediaItem = SHMediaItem(properties: [
    .title: "FoodMath",
    .subtitle: "Count on Me",
    .teacher: "Neil",
    .episode: 3
])

// Add to catalog
let catalog = SHCustomCatalog()
try catalog.addReferenceSignature(signature, representing: [mediaItem])
```

Matching microphone audio against the catalog:
```swift
let session = SHSession(catalog: catalog)
session.delegate = self

let audioEngine = AVAudioEngine()
let inputNode = audioEngine.inputNode
let recordingFormat = inputNode.outputFormat(forBus: 0)

inputNode.installTap(onBus: 0, bufferSize: 8192, format: recordingFormat) { buffer, audioTime in
    session.matchStreamingBuffer(buffer, at: audioTime)
}

try audioEngine.start()
```

Handling a match and computing current offset:
```swift
func session(_ session: SHSession, didFind match: SHMatch) {
    guard let matchedItem = match.mediaItems.first else { return }
    let currentOffset = matchedItem.predictedCurrentMatchOffset

    // Find the last question whose offset is at or before the current position
    let currentQuestion = questions
        .filter { $0.offset <= currentOffset }
        .last

    // Update UI only when the question changes
    if currentQuestion?.id != displayedQuestion?.id {
        displayedQuestion = currentQuestion
        DispatchQueue.main.async { self.updateUI() }
    }
}
```

## Takeaways
- ShazamKit custom catalogs enable on-device, network-free audio synchronization: any audio source can serve as a sync signal by generating a reference signature from it.
- `SHMatchedMediaItem.predictedCurrentMatchOffset` is the key to content synchronization — it provides a continuously updating position estimate in the reference audio without any server calls.
- `matchStreamingBuffer(_:at:)` handles signature generation and query management internally; pass the `AVAudioTime` from the `installTap` block to enable continuity validation.
- Catalogs can embed arbitrary `plist`-compatible metadata via custom `SHMediaItemProperty` keys; there is no schema restriction beyond valid property list value types.
- Distribute custom catalogs as `.shazamcatalog` files (loadable via `SHCustomCatalog.add(from:)`); use `.shazamsignature` files to share individual signatures across tools.

---
_Source: WWDC21 Session 10045 page (abstract, chapter summaries, code samples, and resource links)._
