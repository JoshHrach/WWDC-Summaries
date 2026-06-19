# Create a Great ShazamKit Experience
**WWDC23 · Session 10051** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10051/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10, tvOS 17

## Overview
ShazamKit enables apps to recognize audio from the microphone or prerecorded signatures, matching against Shazam's global music catalog or a custom catalog. This session covers three major 2023 updates: the new `SHManagedSession` that eliminates the need to manually manage `AVAudioEngine` audio buffers; a redesigned Shazam Library API (`SHLibrary`) with read, write, and delete capabilities; and improved matching capabilities including broader PCM format support and multi-match results from custom catalogs.

The session is structured as a code-along, progressively refactoring a "ShazamKit Dance Finder" app from manual `AVAudioEngine` + `SHSession` code to the new `SHManagedSession`, then adding library persistence via `SHLibrary`. Both new classes conform to Swift's `Observable` protocol, enabling zero-boilerplate SwiftUI state observation.

## Key Topics

### SHManagedSession (New)
`SHManagedSession` is a high-level wrapper that handles microphone access requests and audio recording internally. Developers no longer need to configure `AVAudioEngine`, install taps, or manage PCM buffer formats. It supports:
- **Single match**: `await managedSession.result()` — returns a `SHSession.Result` enum with `.match`, `.noMatch`, or `.error` cases.
- **Continuous matching**: `for await result in managedSession.results` — async sequence for long-running sessions.
- **Cancellation**: `managedSession.cancel()`.
- **Preparation**: `await managedSession.prepare()` — preallocates resources and begins prerecording so the first match returns faster.
- **State tracking**: `managedSession.state` — `.idle`, `.prerecording`, or `.matching`. Conforms to `Observable` so SwiftUI views automatically refresh.

Works with AirPods audio; does not support matching arbitrary signature files (use `SHSession` for that).

### SHLibrary (New, replaces SHMediaLibrary)
`SHLibrary.default` provides a per-app view of the Shazam Library:
- **Add**: `try await SHLibrary.default.addItems(mediaItems)`.
- **Read**: `SHLibrary.default.items` — returns only items your app has added, synced across devices.
- **Delete**: `try await SHLibrary.default.removeItems(mediaItems)` — only items your app added.
Also conforms to `Observable`, so `List(SHLibrary.default.items)` in SwiftUI auto-refreshes.

### Expanded PCM Format Support
`SHSession.matchStreamingBuffer` previously required specific sample rates. It now accepts PCM buffers with most format settings across a broad range of sample rates, performing format conversion automatically.

### Multi-Match from Custom Catalogs
When a query signature matches multiple reference signatures in a custom catalog (e.g., a TV show where every episode shares the same intro audio), `SHSession` now returns all matches sorted by quality. Apps can filter `match.mediaItems` to select the appropriate result. Proper metadata annotation of reference signatures enables reliable filtering.

### When to Use SHManagedSession vs. SHSession
| Scenario | Use |
|---|---|
| Microphone recognition | `SHManagedSession` |
| AirPods audio recognition | `SHManagedSession` |
| Arbitrary signature files | `SHSession` |
| Custom PCM buffer pipeline | `SHSession` |
| Specific audio format control | `SHSession` |

## APIs & Frameworks

**ShazamKit**
- `SHManagedSession` **[NEW]** — managed recording + matching session
- `SHManagedSession.init()` / `SHManagedSession(catalog:)` **[NEW]** — initialize with optional custom catalog
- `SHManagedSession.result()` async **[NEW]** — single match attempt
- `SHManagedSession.results` **[NEW]** — async sequence for continuous matching
- `SHManagedSession.cancel()` **[NEW]** — stop matching and recording
- `SHManagedSession.prepare()` async **[NEW]** — preallocate and prerecord for faster first match
- `SHManagedSession.state` **[NEW]** — current session state: `.idle`, `.prerecording`, `.matching`
- `SHSession.Result` enum **[NEW]** — `.match(SHMatch)`, `.noMatch(SHSignature)`, `.error(Error, SHSignature?)`
- `SHLibrary` **[NEW]** — per-app Shazam Library with read/write/delete
- `SHLibrary.default` **[NEW]** — shared library instance
- `SHLibrary.addItems(_:)` async **[NEW]** — write media items to library
- `SHLibrary.items` **[NEW]** — current library items (Observable property)
- `SHLibrary.removeItems(_:)` async **[NEW]** — delete app-added items from library
- `SHSession` — existing session for custom buffer pipelines (updated: now accepts broader PCM formats)
- `SHSession.matchStreamingBuffer(_:at:)` — updated to accept most PCM sample rates **[UPDATED]**
- `SHMatch` — existing match result type
- `SHMatch.mediaItems` — array of `SHMatchedMediaItem`; now returns all multi-catalog matches sorted by quality **[UPDATED]**
- `SHMatchedMediaItem` — match metadata (title, artist, genres, appleMusicURL, etc.)
- `SHCustomCatalog` — custom reference signature catalog
- `SHMediaItem` — media item for custom catalogs
- `SHLibrary` (replaces) `SHMediaLibrary` — legacy write-only library API (superseded)

**Swift / Observation**
- `@Observable` / `Observable` protocol — new Swift macro; both `SHManagedSession` and `SHLibrary` conform, enabling zero-boilerplate SwiftUI observation

## Code Highlights

Single match with SHManagedSession:
```swift
let managedSession = SHManagedSession()
let result = await managedSession.result()
switch result {
case .match(let match): print("Found: \(match.mediaItems.count) items")
case .noMatch: print("No match")
case .error(let error, _): print("Error: \(error)")
}
```

Continuous matching with state-driven SwiftUI:
```swift
for await result in managedSession.results {
    // handle result
}
```

Reading library in SwiftUI (auto-refreshing):
```swift
List(SHLibrary.default.items) { item in
    MediaItemView(item: item)
}
```

## Takeaways
- `SHManagedSession` replaces complex `AVAudioEngine` setup with a single async call; use it for all microphone/AirPods matching scenarios.
- Both `SHManagedSession` and `SHLibrary` adopt Swift `Observable`, making SwiftUI integration seamless without `@Published` or manual state management.
- `SHLibrary` replaces `SHMediaLibrary` with bidirectional access (add, read, delete) and automatic cross-device sync.
- Multi-match from custom catalogs unlocks new use cases like per-episode TV show syncing; proper metadata annotation is essential for filtering results.

---
_Source: WWDC23 Session 10051 page (abstract, chapter summaries, code samples, and resource links)._
