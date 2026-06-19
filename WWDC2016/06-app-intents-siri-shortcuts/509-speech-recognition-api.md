# Speech Recognition API
**WWDC16 · Session 509** · [Watch](https://developer.apple.com/videos/play/wwdc2016/509/)

_Platforms:_ iOS 10+

## Overview
iOS 10 introduces the Speech framework, a brand new API that exposes the same speech recognition engine used by Siri and Keyboard Dictation to third-party apps. Prior to iOS 10, apps wanting speech-to-text had to go through the keyboard dictation flow (available since iOS 5), which offered no developer control over audio recording, language selection, or recognition lifecycle. The new framework corrects all of these limitations.

Recognition is powered by Apple's server-side infrastructure by default, supporting over 50 languages and dialects at launch. Some newer devices support on-device recognition, and the framework provides an availability API (`SFSpeechRecognizer.isAvailable`) to detect capability without manually checking for internet connectivity. Because audio is transmitted to Apple servers, the user must grant explicit permission before any recognition can begin.

## Key Topics

### Authorization
- Speech recognition requires a usage description in `Info.plist` (`NSSpeechRecognitionUsageDescription`).
- `SFSpeechRecognizer.requestAuthorization(_:)` class method prompts the user once; the result is cached in Privacy settings.
- Completion handler does not guarantee a specific execution context — dispatch to main queue for UI updates.
- Authorization status: `.authorized`, `.denied`, `.restricted`, `.notDetermined`.
- Gracefully disable speech features when authorization is not granted; users can change the setting later.

### Pre-Recorded File Recognition
- Create an `SFSpeechURLRecognitionRequest` with the file URL of pre-recorded audio.
- Hand the request to `SFSpeechRecognizer.recognitionTask(with:resultHandler:)`.
- Result handler is called multiple times as recognition is refined; check `result.isFinal` to get the definitive transcription.

### Live Audio Recognition
- Create an `SFSpeechAudioBufferRecognitionRequest`.
- Use `AVAudioEngine` to tap the input node; append audio buffers to the request with `append(_:)`.
- Audio buffers can be appended before or after calling `recognitionTask(with:resultHandler:)`.
- When recording ends, call `endAudio()` on the request to signal that no more audio is coming.
- Hold the returned `SFSpeechRecognitionTask` and call `cancel()` if the user cancels or recording is interrupted.

### Recognition Results
- `SFSpeechRecognitionResult` contains:
  - `bestTranscription` — highest-confidence hypothesis.
  - `transcriptions` — array of alternative hypotheses.
  - `isFinal` — signals the terminal result.
- `SFTranscription` holds an array of `SFTranscriptionSegment` objects, each with:
  - `substring` — recognized word or phrase.
  - `timestamp` and `duration` — timing information.
  - `confidence` — per-segment confidence value.
  - `alternativeSubstrings` — alternative words at this position.

### Availability and Rate Limits
- `SFSpeechRecognizer.isAvailable` — check before starting recognition (changes with connectivity).
- `SFSpeechRecognizer(locale:)` — failable; returns nil if locale is unsupported.
- Per-device daily recognition limits exist; apps may also be throttled globally per request-per-day.
- Treat like other service-backed APIs (e.g., CLGeocoder): handle rate limiting and network errors gracefully.
- iOS 10 enforces a strict audio duration limit of approximately one minute (matching keyboard dictation).

### Privacy and Best Practices
- Display a recording indicator (visual or audio) when capturing the user's speech.
- Do not send sensitive speech (passwords, health data, financial information) to speech recognition.
- Display recognized text in real time so users can see and correct recognition errors.
- Privacy setting label: **NSSpeechRecognitionUsageDescription**.

## APIs & Frameworks

- **Speech** framework [NEW] — top-level import for all speech recognition types
- `SFSpeechRecognizer` [NEW] — main recognizer class; one per language
  - `init(locale: Locale)` [NEW] — failable; returns nil for unsupported locales
  - `init()` [NEW] — uses device's current locale
  - `class func requestAuthorization(_ handler: (SFSpeechRecognizerAuthorizationStatus) -> Void)` [NEW]
  - `var isAvailable: Bool` [NEW] — changes dynamically with connectivity; KVO-observable
  - `func recognitionTask(with: SFSpeechRecognitionRequest, resultHandler: (SFSpeechRecognitionResult?, Error?) -> Void) -> SFSpeechRecognitionTask` [NEW]
  - `var delegate: SFSpeechRecognizerDelegate?` [NEW]
- `SFSpeechRecognizerAuthorizationStatus` [NEW] — enum: `.authorized`, `.denied`, `.restricted`, `.notDetermined`
- `SFSpeechRecognizerDelegate` [NEW] — `speechRecognizer(_:availabilityDidChange:)` callback
- `SFSpeechRecognitionRequest` [NEW] — abstract base class for recognition requests
  - `var shouldReportPartialResults: Bool` [NEW] — enable incremental results
  - `var contextualStrings: [String]` [NEW] — hints to improve recognition of domain-specific words
  - `var interactionIdentifier: String?` [NEW] — groups related tasks
  - `var taskHint: SFSpeechRecognitionTaskHint` [NEW] — `.unspecified`, `.dictation`, `.search`, `.confirmation`
- `SFSpeechURLRecognitionRequest` [NEW] — file-based request; `init(url: URL)` [NEW]
- `SFSpeechAudioBufferRecognitionRequest` [NEW] — live audio request
  - `func append(_ audioPCMBuffer: AVAudioPCMBuffer)` [NEW]
  - `func endAudio()` [NEW]
- `SFSpeechRecognitionTask` [NEW] — handle for monitoring and controlling a recognition task
  - `var state: SFSpeechRecognitionTaskState` [NEW]
  - `var isCancelled: Bool` [NEW]
  - `var isFinishing: Bool` [NEW]
  - `func cancel()` [NEW]
  - `func finish()` [NEW]
- `SFSpeechRecognitionResult` [NEW]
  - `var bestTranscription: SFTranscription` [NEW]
  - `var transcriptions: [SFTranscription]` [NEW]
  - `var isFinal: Bool` [NEW]
- `SFTranscription` [NEW]
  - `var formattedString: String` [NEW]
  - `var segments: [SFTranscriptionSegment]` [NEW]
- `SFTranscriptionSegment` [NEW]
  - `var substring: String` [NEW]
  - `var substringRange: NSRange` [NEW]
  - `var timestamp: TimeInterval` [NEW]
  - `var duration: TimeInterval` [NEW]
  - `var confidence: Float` [NEW]
  - `var alternativeSubstrings: [String]` [NEW]
- `AVAudioEngine` — used to obtain live audio buffers for SFSpeechAudioBufferRecognitionRequest
- `AVAudioInputNode` — tapped for live microphone audio
- `NSSpeechRecognitionUsageDescription` — required Info.plist key [NEW in iOS 10]

## Code Highlights

Request authorization and handle status:
```swift
SFSpeechRecognizer.requestAuthorization { authStatus in
    OperationQueue.main.addOperation {
        switch authStatus {
        case .authorized:
            self.microphoneButton.isEnabled = true
        default:
            self.microphoneButton.isEnabled = false
        }
    }
}
```

Recognize a pre-recorded file:
```swift
let recognizer = SFSpeechRecognizer()
guard let recognizer = recognizer, recognizer.isAvailable else { return }
let request = SFSpeechURLRecognitionRequest(url: audioFileURL)
recognizer.recognitionTask(with: request) { result, error in
    guard let result = result else { print("Error: \(error!)"); return }
    if result.isFinal {
        print(result.bestTranscription.formattedString)
    }
}
```

Recognize live audio using AVAudioEngine:
```swift
let request = SFSpeechAudioBufferRecognitionRequest()
let inputNode = audioEngine.inputNode
recognitionTask = recognizer.recognitionTask(with: request) { result, error in
    if let result = result, result.isFinal { /* handle */ }
}
let recordingFormat = inputNode.outputFormat(forBus: 0)
inputNode.installTap(onBus: 0, bufferSize: 1024, format: recordingFormat) { buffer, _ in
    request.append(buffer)
}
audioEngine.prepare(); try audioEngine.start()
// When done:
request.endAudio()
// On cancel:
recognitionTask?.cancel()
```

## Takeaways
- The Speech framework gives third-party apps access to the same high-quality, server-side speech recognition used by Siri and Keyboard Dictation, without requiring any user data collection.
- Both file-based and live audio recognition are supported via `SFSpeechURLRecognitionRequest` and `SFSpeechAudioBufferRecognitionRequest`; the result handler fires incrementally and signals completion with `isFinal`.
- Results include not just transcription text but alternative hypotheses, per-word confidence scores, and timing data — enabling richer UI experiences.
- The framework is rate-limited per device and per app per day; always handle `.unavailable` state and network/rate-limit errors gracefully, just as you would with `CLGeocoder` or other service-backed APIs.

---
_Source: WWDC16 Session 509 page (abstract, transcript, and resource links)._
