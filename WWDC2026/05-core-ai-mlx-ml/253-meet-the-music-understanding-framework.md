# Meet the Music Understanding Framework
**WWDC26 · Session 253** · [Watch](https://developer.apple.com/videos/play/wwdc2026/253/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, visionOS, watchOS

## Overview
Music Understanding is a new Apple framework that analyzes audio across six musical dimensions entirely on-device: key, rhythm, structure, pace, instrument activity, and loudness. Analysis works offline and does not require network access. The framework supports both file-based batch analysis via `AVURLAsset` and streaming real-time analysis via a custom `AudioProvider` protocol.

The session introduces the framework's design through a walk of the `SessionResult` type hierarchy, demonstrating each of the six result types with their constituent data structures. A companion sample app — the Music Understanding Lab — provides a visual representation of every analysis type, serving as both a reference implementation and an exploratory tool.

## Key Topics

### Framework Integration
Initialize a `MusicUnderstandingSession` with either an `AVURLAsset` (for file analysis) or an `AudioProvider` (for streaming). Call `session.analyze()` to run all six analyses, or `session.analyze(for:)` with a specific set. The resulting `SessionResult` is `Codable` and `Sendable`, making it easy to cache and transfer.

### Key Analysis
`KeyResult` contains an array of `RangedValue<KeySignature>` — each segment of the audio mapped to a `KeySignature` with a `Tonic` (18 values: A, B-flat, C-sharp, etc.) and `Mode` (`.major` or `.minor`).

### Rhythm Analysis
`RhythmResult` provides arrays of beat and bar onset times as `CMTime` values, plus an optional `beatsPerMinute: Float?` for the overall tempo.

### Structure Analysis
`StructureResult` provides three levels of musical segmentation as `CMTimeRange` arrays: `sections` (large-scale blocks), `segments` (mid-level), and `phrases` (fine-grained melodic units).

### Pace Analysis
`PaceResult` contains `RangedValue<Double>` ranges — the pace value is a cuts-per-minute suggestion for video editing applications. The suggested formula is `timePerClip = 60 / paceValue`.

### Instrument Activity Analysis
`InstrumentActivityResult` provides per-instrument analysis: `ranges` (active time ranges per `Instrument`) and `activity` (continuous activity level as `TimedValue<Float>` for precise intensity over time).

### Loudness Analysis
`LoudnessResult` provides integrated, momentary, short-term, and peak loudness measurements as `TimedValue<Float>`, following standard loudness metering conventions (LUFS-style). Loudness also supports a streaming API via `session.loudnessResults` — an `AsyncSequence` — for real-time level metering.

### Streaming API and AudioProvider
For live or custom audio sources, implement `AudioProvider` as an `AsyncSequence` of `AVReadOnlyAudioPCMBuffer` values. Use `withThrowingTaskGroup` to concurrently drive the audio into the session and consume streaming results.

### Serialization
`SessionResult` is `Codable` — encode to JSON with `JSONEncoder().encode(results)` for caching, sharing, or further processing.

## APIs & Frameworks

**MusicUnderstanding** **[NEW]**
- `MusicUnderstandingSession` — main analysis session
  - `init(asset:)` — initialize from `AVURLAsset`
  - `init(audioProvider:)` — initialize from a custom audio provider
  - `analyze()` — analyze all six dimensions; returns `SessionResult`
  - `analyze(for:)` — analyze specific dimensions (e.g., `[.loudness]`)
  - `loudnessResults: AsyncSequence<LoudnessResult, Error>` — streaming loudness **[NEW]**
- `SessionResult` — container for all analysis results; `Codable`, `Sendable`
  - `.key: KeyResult?`
  - `.rhythm: RhythmResult?`
  - `.structure: StructureResult?`
  - `.pace: PaceResult?`
  - `.instrumentActivity: InstrumentActivityResult?`
  - `.loudness: LoudnessResult?`
- `KeyResult` — key analysis result
  - `.ranges: [RangedValue<KeySignature>]`
- `KeySignature` — `Codable`, `Hashable`, `Sendable`
  - `.tonic: Tonic` — `@frozen enum` with 18 pitch values (aFlat, aSharp, a, bFlat, b, c, cSharp, d, dFlat, dSharp, eFlat, e, f, fSharp, g, gFlat, gSharp)
  - `.mode: Mode` — `.major` or `.minor`
- `RhythmResult`
  - `.beats: [CMTime]` — beat onset times
  - `.bars: [CMTime]` — bar (measure) onset times
  - `.beatsPerMinute: Float?` — overall tempo
- `StructureResult`
  - `.sections: [CMTimeRange]`
  - `.segments: [CMTimeRange]`
  - `.phrases: [CMTimeRange]`
- `PaceResult`
  - `.ranges: [RangedValue<Double>]` — pace values in cuts-per-minute
- `InstrumentActivityResult`
  - `.ranges: [Instrument: [CMTimeRange]]` — active time ranges per instrument
  - `.activity: [Instrument: [TimedValue<Float>]]` — continuous activity level
- `Instrument` — enum of recognized instrument categories
- `LoudnessResult`
  - `.integrated: TimedValue<Float>` — integrated loudness (LUFS-style)
  - `.momentary: [TimedValue<Float>]` — 400ms window measurements
  - `.shortTerm: [TimedValue<Float>]` — 3s window measurements
  - `.peak: TimedValue<Float>` — true peak level
- `TimedValue<Value>` — value paired with a `CMTime` timestamp; `Codable`, `Equatable`, `Sendable`
- `RangedValue<Value>` — value paired with a `CMTimeRange`; `Codable`, `Equatable`, `Sendable`
- `AudioProvider` protocol — `AsyncSequence` + `AsyncIteratorProtocol` of `AVReadOnlyAudioPCMBuffer?`

**AVFoundation**
- `AVURLAsset` — audio file asset (with `AVURLAssetPreferPreciseDurationAndTimingKey: true`)
- `AVReadOnlyAudioPCMBuffer` — audio buffer type for streaming
- `CMTime`, `CMTimeRange` — Core Media time types used throughout result types

**Related Resources**
- [MusicUnderstanding documentation](https://developer.apple.com/documentation/MusicUnderstanding)
- [Creating visuals with Music Understanding analysis results](https://developer.apple.com/documentation/MusicUnderstanding/create-visuals-using-musicunderstanding-analysis-results)

## Code Highlights

Batch analysis from a file:
```swift
import MusicUnderstanding
let asset = AVURLAsset(url: url, options: [AVURLAssetPreferPreciseDurationAndTimingKey: true])
let session = try await MusicUnderstandingSession(asset: asset)
let results = try await session.analyze()
// results.key, results.rhythm, results.structure, etc.
```

Streaming loudness with a custom AudioProvider:
```swift
let audioProvider = AudioProvider()
let session = MusicUnderstandingSession(audioProvider: audioProvider)
await withThrowingTaskGroup(of: Void.self) { group in
    group.addTask {
        for try await result in await session.loudnessResults {
            updateAudioLevel(result.momentary.value)
        }
    }
    group.addTask { try await session.analyze(for: [.loudness]) }
}
```

Video editing pace calculation:
```swift
let timePerClip = 60 / paceValue
```

## Takeaways
- Music Understanding provides production-quality on-device audio analysis with zero network dependency — it's appropriate for any app dealing with music, podcasts, or audio content.
- The `loudnessResults` async sequence enables real-time level metering for visualizers, DJ apps, and audio tools without waiting for full file analysis.
- `SessionResult` is `Codable` — analyze once, cache the result as JSON, and reuse it without re-analyzing (especially important for long tracks).
- Use `PaceResult` for video editing features: the cuts-per-minute pace value maps directly to clip duration, making beat-matched video editing trivial to implement.

---
_Source: WWDC26 Session 253 page (abstract, chapter summaries, code samples, and resource links)._
