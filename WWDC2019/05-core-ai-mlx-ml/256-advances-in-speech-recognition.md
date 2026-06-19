# Advances in Speech Recognition
**WWDC19 · Session 256** · [Watch](https://developer.apple.com/videos/play/wwdc2019/256/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15

## Overview
iOS 13 and macOS Catalina bring three significant advances to the Speech framework: macOS support (AppKit and Catalyst/iPad apps), on-device recognition mode for privacy-sensitive applications, and new voice analytics results including speaking rate, average pause duration, jitter, shimmer, pitch, and voicing metrics.

On-device recognition eliminates the network dependency and per-request server limits, at the cost of slightly lower accuracy and a smaller set of supported languages. Server-based recognition now automatically falls back to on-device when the server is unreachable if the device supports it.

## Key Topics

### macOS Support **[NEW]**
- `Speech` framework is now available on macOS Catalina, supporting both AppKit and iPad apps running on Mac.
- Over 50 languages supported, matching iOS.
- Requires microphone permission (`NSMicrophoneUsageDescription`) and Siri enabled by the user.
- API surface is identical to iOS — no macOS-specific changes needed.

### On-Device Recognition **[NEW]**
- `SFSpeechRecognitionRequest.requiresOnDeviceRecognition = true` **[NEW]** — forces recognition to run entirely on device; no audio is sent to Apple servers.
- Before setting this flag, check `SFSpeechRecognizer.supportsOnDeviceRecognition` **[NEW]** to confirm the device/language combination supports on-device mode.
- Supported hardware: iPhone and iPad with Apple A9 processor or later; all Mac models.
- Supported languages: 10+ languages in on-device mode (fewer than the 50+ server supports).
- Privacy advantages: user data never leaves device; no network connection required; no cellular data consumed.
- No per-request or per-audio-duration limits (server mode has limits).
- Accuracy tradeoff: server recognition may be more accurate due to continuous server-side learning; on-device accuracy is still good.
- Automatic fallback: when `requiresOnDeviceRecognition` is `false` (default), if the server is unavailable the system automatically falls back to on-device recognition if supported.

### Voice Analytics **[NEW]**
New acoustic analysis metrics added to `SFTranscription` and `SFTranscriptionSegment`:

- **Speaking rate** — words per minute; available on `SFTranscription.speakingRate`.
- **Average pause duration** — mean length of silence between words; available on `SFTranscription.averagePauseDuration`.
- **Voice analytics features** (per segment, `SFTranscriptionSegment.voiceAnalytics`) — four measures:
  - **Jitter** (`SFVoiceAnalytics.jitter`) — pitch variation expressed as a percentage; elevated jitter may indicate vocal stress or pathology.
  - **Shimmer** (`SFVoiceAnalytics.shimmer`) — amplitude variation expressed in decibels; correlates with voice roughness.
  - **Pitch** (`SFVoiceAnalytics.pitch`) — fundamental frequency; varies by speaker (women and children typically higher).
  - **Voicing** (`SFVoiceAnalytics.voicing`) — identifies voiced vs. unvoiced regions in speech.
- Analytics are individual/context-dependent — values vary with fatigue, conversational context, and speaker characteristics.
- Available periodically during recognition and guaranteed at the final result (`isFinal == true`).

## APIs & Frameworks

**Speech**
- `SFSpeechRecognizer` — existing class; now available on macOS **[NEW]**
- `SFSpeechRecognizer.supportsOnDeviceRecognition: Bool` **[NEW]** — query on-device support for current locale
- `SFSpeechRecognitionRequest.requiresOnDeviceRecognition: Bool` **[NEW]** — opt into on-device-only recognition
- `SFSpeechRecognitionResult` — existing; extended with:
  - `SFTranscription.speakingRate: Double` **[NEW]** — words per minute
  - `SFTranscription.averagePauseDuration: TimeInterval` **[NEW]** — mean pause between words
- `SFTranscriptionSegment.voiceAnalytics: SFVoiceAnalytics?` **[NEW]**
- `SFVoiceAnalytics` **[NEW]** — voice analytics container:
  - `SFVoiceAnalytics.jitter: SFAcousticFeature` **[NEW]** — pitch perturbation (%)
  - `SFVoiceAnalytics.shimmer: SFAcousticFeature` **[NEW]** — amplitude perturbation (dB)
  - `SFVoiceAnalytics.pitch: SFAcousticFeature` **[NEW]** — fundamental frequency
  - `SFVoiceAnalytics.voicing: SFAcousticFeature` **[NEW]** — voiced/unvoiced classification
- `SFAcousticFeature` **[NEW]** — `acousticFeatureValuePerFrame: [Double]`, `frameDuration: TimeInterval`

## Code Highlights

```swift
import Speech

// Check on-device support and request on-device recognition
let recognizer = SFSpeechRecognizer(locale: Locale(identifier: "en-US"))!
guard recognizer.isAvailable else { return }

let request = SFSpeechURLRecognitionRequest(url: audioFileURL)

if recognizer.supportsOnDeviceRecognition {
    request.requiresOnDeviceRecognition = true  // no data sent to Apple servers
}

recognizer.recognitionTask(with: request) { result, error in
    guard let result else { return }

    // Speaking rate and pause analytics (available on final result and periodically)
    let transcription = result.bestTranscription
    print("Speaking rate: \(transcription.speakingRate) wpm")
    print("Avg pause: \(transcription.averagePauseDuration)s")

    // Voice analytics per segment
    for segment in transcription.segments {
        if let analytics = segment.voiceAnalytics {
            print("Jitter: \(analytics.jitter.acousticFeatureValuePerFrame)")
            print("Shimmer: \(analytics.shimmer.acousticFeatureValuePerFrame)")
            print("Pitch: \(analytics.pitch.acousticFeatureValuePerFrame)")
            print("Voicing: \(analytics.voicing.acousticFeatureValuePerFrame)")
        }
    }

    if result.isFinal {
        print("Final transcript: \(transcription.formattedString)")
    }
}
```

```swift
// Live microphone recognition with on-device mode
let audioEngine = AVAudioEngine()
let request = SFSpeechAudioBufferRecognitionRequest()
request.requiresOnDeviceRecognition = recognizer.supportsOnDeviceRecognition

let inputNode = audioEngine.inputNode
let format = inputNode.outputFormat(forBus: 0)
inputNode.installTap(onBus: 0, bufferSize: 1024, format: format) { buffer, _ in
    request.append(buffer)
}
audioEngine.prepare()
try audioEngine.start()

recognizer.recognitionTask(with: request) { result, error in
    if let result {
        print(result.bestTranscription.formattedString)
    }
}
```

## Takeaways
- Set `requiresOnDeviceRecognition = true` for any use case involving sensitive user data (medical, financial, personal notes) — audio never leaves the device and there are no server rate limits.
- Always check `supportsOnDeviceRecognition` before setting the flag; gracefully fall back to server mode if unsupported.
- `speakingRate` and `averagePauseDuration` are useful for presentation coaching, accessibility features, and conversational UI — available with no additional setup beyond the existing recognition task.
- Voice analytics (`jitter`, `shimmer`, `pitch`, `voicing`) open the door to vocal health monitoring, emotion-aware apps, and accessibility tools, but values are highly individual — calibrate per user.
- Speech framework on macOS Catalina is API-identical to iOS; existing iOS recognition code compiles and runs on Mac unchanged.

---
_Source: WWDC19 Session 256 page (transcript, abstract, and resource links)._
