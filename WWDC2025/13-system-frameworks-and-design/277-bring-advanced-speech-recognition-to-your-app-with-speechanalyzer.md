# Bring Advanced Speech Recognition to Your App with SpeechAnalyzer
**WWDC25 · Session 277** · [Watch](https://developer.apple.com/videos/play/wwdc2025/277/)

_Platforms:_ iOS 26, macOS Tahoe 26, iPadOS 26

## Overview
This session introduces the redesigned Speech framework for iOS 26 and macOS Tahoe, centered on the new `SpeechAnalyzer` API. Apple is replacing the legacy `SFSpeechRecognizer` with a composable, modern architecture that separates audio input analysis from individual recognition tasks (called "modules"). The headline module is `SpeechTranscriber`, which provides on-device, real-time transcription with support for volatile (partial) results, audio time-range attribution, and multiple locales — all with explicit asset lifecycle management.

The session covers two primary scenarios: offline dictation-style transcription and progressive live transcription (for continuous input like voice notes or real-time captions).

## Key Topics

### SpeechAnalyzer and Module Architecture
`SpeechAnalyzer` is the new entry point for all speech analysis. It accepts an audio format and a list of modules (e.g., `SpeechTranscriber`), then consumes an async stream of `AnalyzerInput` chunks. Multiple modules can share the same audio pipeline, avoiding redundant processing.

### SpeechTranscriber
`SpeechTranscriber` is the transcription module. It is initialized with a locale, transcription options (e.g., enable/disable volatile results), reporting options (whether to report interim results), and attribute options (e.g., attach audio time ranges to result runs). Each recognition result carries a `.text` property typed as `AttributedString`, enabling rich metadata like timing or confidence as string attributes.

### Asset Management
Language models must be downloaded before use. `AssetInventory` is the interface for querying installed locales and requesting downloads. `AssetInventory.assetInstallationRequest()` returns a request object; calling `.downloadAndInstall()` on it fetches the necessary model assets. `deallocate(locale:)` releases them when no longer needed. The API surface also includes `SpeechTranscriber.supportedLocales` and `.installedLocales` for discovery.

### Streaming Input
`SpeechAnalyzer` accepts audio via `AsyncStream<AnalyzerInput>`. The app creates a stream using `AsyncStream.makeStream()`, then feeds `AnalyzerInput` values into the continuation. `analyzeSequence(from:)` begins processing; finalization is controlled via `finalizeAndFinishThroughEndOfInput()` (processes remaining audio) or `cancelAndFinishNow()` (terminates immediately).

### Result Structure
Each result emitted by the analyzer has:
- `.text` — `AttributedString` containing the transcribed text with optional per-run attributes (e.g., `CMTimeRange` on each word run when `.audioTimeRange` attribute option is enabled)
- `.isFinal` — Bool indicating whether this is the final transcription for that segment
- `.audioTimeRange` — The CMTimeRange span covered by this result

### Presets
Two high-level presets simplify configuration: `.offlineTranscription` (batch, high-accuracy, not streaming) and `.progressiveLiveTranscription` (low-latency, suitable for real-time display).

## APIs & Frameworks

**Speech Framework (iOS 26, macOS Tahoe 26)**
- **[NEW]** `SpeechAnalyzer` — main analysis entry point; replaces `SFSpeechRecognizer`
- **[NEW]** `SpeechTranscriber` — transcription module; composable with SpeechAnalyzer
- **[NEW]** `SpeechTranscriber(locale:transcriptionOptions:reportingOptions:attributeOptions:)`
- **[NEW]** `SpeechTranscriber.supportedLocales` — all supported locales
- **[NEW]** `SpeechTranscriber.installedLocales` — currently downloaded locales
- **[NEW]** `DictationTranscriber` — specialized preset for dictation use cases
- **[NEW]** `AssetInventory` — manages downloadable language model assets
- **[NEW]** `AssetInventory.assetInstallationRequest()` — creates a download request
- **[NEW]** `downloadAndInstall()` on installation request
- **[NEW]** `deallocate(locale:)` — releases model assets
- **[NEW]** `analyzeSequence(from:)` — begin streaming audio analysis
- **[NEW]** `finalizeAndFinishThroughEndOfInput()` — graceful shutdown (finishes processing remaining audio)
- **[NEW]** `cancelAndFinishNow()` — immediate termination
- **[NEW]** `AnalyzerInput` — typed audio chunk for analyzer input stream
- **[NEW]** `SpeechAnalyzer.bestAvailableAudioFormat(compatibleWith:)` — query optimal PCM format
- **[NEW]** Transcription result: `.text: AttributedString`, `.isFinal: Bool`, `.audioTimeRange: CMTimeRange`
- **[NEW]** Reporting options: `.volatileResults` (partial/interim results)
- **[NEW]** Attribute options: `.audioTimeRange` (per-run timing data on AttributedString)
- **[NEW]** Presets: `.offlineTranscription`, `.progressiveLiveTranscription`

**Core Media**
- `CMTimeRange` — used on `AttributedString` runs for per-word timing metadata

**Foundation**
- `AttributedString` — returned as `.text` on each transcription result; carries per-run timing attributes

## Code Highlights
Initialize and configure the analyzer:
```swift
let transcriber = SpeechTranscriber(
    locale: Locale(identifier: "en-US"),
    transcriptionOptions: [],
    reportingOptions: [.volatileResults],
    attributeOptions: [.audioTimeRange]
)
let analyzer = SpeechAnalyzer(modules: [transcriber])
```

Stream audio and collect results:
```swift
let (stream, continuation) = AsyncStream<AnalyzerInput>.makeStream()
Task {
    for try await result in analyzer.analyzeSequence(from: stream) {
        let text: AttributedString = result.text
        let isFinal = result.isFinal
    }
}
// Feed audio chunks:
continuation.yield(AnalyzerInput(audioBuffer))
// Finalize:
await analyzer.finalizeAndFinishThroughEndOfInput()
```

Query and download a locale asset:
```swift
let request = await AssetInventory.assetInstallationRequest(for: transcriber, locale: .current)
try await request.downloadAndInstall()
```

## Takeaways
- Migrate from `SFSpeechRecognizer` to `SpeechAnalyzer` + `SpeechTranscriber` for a modern, on-device, composable speech pipeline.
- Use `AssetInventory` to explicitly manage locale model downloads; check `.installedLocales` before creating an analyzer session.
- Enable `.volatileResults` for real-time streaming captions; enable `.audioTimeRange` attribute option to get per-word timing on the `AttributedString` result for karaoke-style highlighting.
- Use `.progressiveLiveTranscription` preset for low-latency real-time transcription; use `.offlineTranscription` preset for high-accuracy batch scenarios.

---
_Source: WWDC25 Session 277 page (abstract, chapter summaries, code samples, and resource links)._
