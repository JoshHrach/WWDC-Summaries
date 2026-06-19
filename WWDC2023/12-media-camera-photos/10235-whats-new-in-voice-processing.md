# What's new in voice processing
**WWDC23 · Session 10235** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10235/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, tvOS 17

## Overview
Apple's voice processing APIs — `AUVoiceIO` (AUVoiceProcessingIO) and `AVAudioEngine` in voice processing mode — gain two new capabilities in 2023: configurable ducking of other audio and muted talker detection. Both APIs provide the same echo cancellation, noise suppression, automatic gain control, and mic mode support (Standard, Voice Isolation, Wide Spectrum) that power FaceTime and the Phone app.

Voice processing is also now available on tvOS 17 for the first time (via Continuity Camera). Additionally, macOS developers who have not yet adopted the voice processing APIs can use new Core Audio HAL APIs for on-device voice activity detection.

## Key Topics

**Other Audio Ducking**
- "Other audio" = any audio stream besides the voice processing output (other app audio, in-app music, etc.)
- Previously a fixed ducking amount; now configurable per app
- Two independent controls: ducking style (`mEnableAdvancedDucking`) and ducking amount (`mDuckingLevel`)
- `mEnableAdvancedDucking`: when enabled, dynamically adjusts based on voice activity — more ducking when either party speaks, less when silent (similar to FaceTime SharePlay behavior)
- `mDuckingLevel`: `.default` (unchanged from before), `.min` (loudest other audio), `.mid`, `.max` (most ducking, best voice intelligibility)
- Available on both `AUVoiceIO` and `AVAudioEngine`

**Muted Talker Detection**
- Detects when the local user speaks while muted, triggering a notification so the app can prompt them to unmute
- First introduced in iOS 15; now available on macOS 14 and tvOS 17 **[NEW platforms]**
- Requires muting through the voice processing mute API (not another mechanism)
- Listener block is invoked only on state changes (started/stopped), not continuously
- Available on both `AUVoiceIO` and `AVAudioEngine`

**Voice Activity Detection via Core Audio HAL (macOS only)**
- Alternative for apps not yet using the voice processing APIs
- Works on echo-cancelled microphone input
- Two new HAL properties:
  - `kAudioDevicePropertyVoiceActivityDetectionEnable` — enable detection
  - `kAudioDevicePropertyVoiceActivityDetectionState` — property listener for state changes
- Detection works regardless of process mute state; app must combine with mute state logic
- Recommend using HAL process mute API to suppress recording indicator

**tvOS Support**
- Voice processing APIs now available on tvOS 17 via Continuity Camera
- Same API as iOS/macOS; see "Discover Continuity Camera for tvOS"

## APIs & Frameworks

**AUVoiceIO / AudioUnit (C)**
- `AUVoiceIOOtherAudioDuckingConfiguration` **[NEW]** struct:
  - `mEnableAdvancedDucking: Boolean`
  - `mDuckingLevel: AUVoiceIOOtherAudioDuckingLevel`
- `AUVoiceIOOtherAudioDuckingLevel` **[NEW]** enum: `kAUVoiceIOOtherAudioDuckingLevelDefault` (0), `kAUVoiceIOOtherAudioDuckingLevelMin` (10), `kAUVoiceIOOtherAudioDuckingLevelMid` (20), `kAUVoiceIOOtherAudioDuckingLevelMax` (30)
- `kAUVoiceIOProperty_OtherAudioDuckingConfiguration` **[NEW]** — AudioUnit property key
- `AUVoiceIOMutedSpeechActivityEventListener` **[NEW]** — block type for muted talker callback
- `AUVoiceIOMutedSpeechActivityEvent` **[NEW]** — enum: `kAUVoiceIOSpeechActivityHasStarted`, `kAUVoiceIOSpeechActivityHasEnded`
- `kAUVoiceIOProperty_MutedSpeechActivityEventListener` **[NEW]** — AudioUnit property key to register listener
- `kAUVoiceIOProperty_MuteOutput` — existing mute property (required for muted talker detection)

**AVAudioEngine (Swift/ObjC)**
- `AVAudioVoiceProcessingOtherAudioDuckingConfiguration` **[NEW]** struct:
  - `enableAdvancedDucking: ObjCBool`
  - `duckingLevel: AVAudioVoiceProcessingOtherAudioDuckingConfiguration.Level`
- `AVAudioVoiceProcessingOtherAudioDuckingConfiguration.Level` **[NEW]** enum: `.default`, `.min`, `.mid`, `.max`
- `AVAudioInputNode.voiceProcessingOtherAudioDuckingConfiguration` **[NEW]** — set on input node
- `AVAudioVoiceProcessingSpeechActivityEvent` **[NEW]** enum: `.started`, `.ended`
- `AVAudioInputNode.setMutedSpeechActivityEventListener(_:)` **[NEW]**
- `AVAudioInputNode.isVoiceProcessingInputMuted` — existing mute property (required)
- `AVAudioInputNode.setVoiceProcessingEnabled(_:)` — existing method

**Core Audio HAL (macOS only)**
- `kAudioDevicePropertyVoiceActivityDetectionEnable` **[NEW]** — set on input device scope
- `kAudioDevicePropertyVoiceActivityDetectionState` **[NEW]** — register `AudioObjectPropertyListenerProc` for state changes
- `AudioObjectSetPropertyData`, `AudioObjectAddPropertyListener`, `AudioObjectGetPropertyData` — existing HAL APIs

## Code Highlights

Setting ducking configuration (AVAudioEngine):
```swift
try inputNode.setVoiceProcessingEnabled(true)
let duckingConfig = AVAudioVoiceProcessingOtherAudioDuckingConfiguration(
    mEnableAdvancedDucking: true,
    mDuckingLevel: .min)
inputNode.voiceProcessingOtherAudioDuckingConfiguration = duckingConfig
```

Muted talker detection (AVAudioEngine):
```swift
let listener = { (event: AVAudioVoiceProcessingSpeechActivityEvent) in
    if event == .started {
        // Prompt user to unmute
    } else if event == .ended {
        // User stopped talking while muted
    }
}
inputNode.setMutedSpeechActivityEventListener(listener)
// When user mutes:
inputNode.isVoiceProcessingInputMuted = true
```

Muted talker detection (AUVoiceIO, C):
```c
AUVoiceIOMutedSpeechActivityEventListener listener = ^(AUVoiceIOMutedSpeechActivityEvent event) {
    if (event == kAUVoiceIOSpeechActivityHasStarted) {
        // Prompt user to unmute
    }
};
AudioUnitSetProperty(auVoiceIO, kAUVoiceIOProperty_MutedSpeechActivityEventListener,
    kAudioUnitScope_Global, 0, &listener, sizeof(listener));
```

## Takeaways
- If your VoIP app plays background music or other audio alongside voice, tune `mDuckingLevel` and consider enabling advanced ducking to dynamically match FaceTime's SharePlay behavior.
- Add muted talker detection to eliminate the "you were muted" awkwardness — the API is now unified across iOS, macOS, and tvOS.
- macOS apps not yet using the voice processing stack can use `kAudioDevicePropertyVoiceActivityDetectionEnable` + `kAudioDevicePropertyVoiceActivityDetectionState` as a lightweight on-device alternative.
- Always mute via the framework's own mute API — external muting prevents the muted talker detector from firing.

---
_Source: WWDC23 Session 10235 page (abstract, chapter summaries, code samples, and resource links)._
