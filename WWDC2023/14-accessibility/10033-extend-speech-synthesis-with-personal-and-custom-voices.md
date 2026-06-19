# Extend Speech Synthesis with Personal and Custom Voices
**WWDC23 · Session 10033** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10033/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, watchOS 10

## Overview
This session covers three major advancements in speech synthesis on Apple platforms: SSML (Speech Synthesis Markup Language) support, a new Speech Synthesis Provider extension architecture that lets third-party developers bring custom synthesizers and voices system-wide, and Personal Voice — a new feature that lets users record and recreate their own voice on-device for use in augmentative and alternative communication (AAC) apps.

The Speech Synthesis Provider extension is built on Audio Unit Extensions, allowing a custom synthesizer to run in its own process rather than the host app. It receives SSML input and renders audio buffers, while the system manages all audio session and playback concerns. Voices registered through this extension appear alongside system voices across all apps, including VoiceOver.

Personal Voice is generated entirely on-device from user recordings and appears in the standard `AVSpeechSynthesisVoice.speechVoices()` list. Apps that serve AAC use cases can request authorization to use it, giving people who may lose their voice the ability to speak with their own synthesized voice.

## Key Topics

### SSML (Speech Synthesis Markup Language)
- W3C standard for declarative representation of spoken text using XML.
- Supports control of rate, pitch, pause duration, and other prosodic properties.
- Used by first-party synthesizers including WebSpeech in WebKit.
- `AVSpeechUtterance(ssmlRepresentation:)` initializer creates utterances directly from SSML strings.

### Speech Synthesis Provider Extension
- New Audio Unit Extension type: "Speech Synthesizer" audio unit.
- Extension runs out-of-process; no audio session management required by the developer.
- Host app manages voice purchasing and stores available voices in a shared `UserDefaults` App Group.
- Extension overrides `speechVoices` getter, `synthesizeSpeechRequest(_:)`, `cancelSpeechRequest()`, and `internalRenderBlock`.
- `AVSpeechSynthesisProviderVoice.updateSpeechVoices()` signals to the system that the voice list has changed.
- `AVSpeechSynthesizer.availableVoicesDidChangeNotification` lets any app observe system voice list changes.

### Personal Voice
- Users record their voice in Settings; synthesis model generated on-device.
- Personal Voice appears in `AVSpeechSynthesisVoice.speechVoices()`, identified by the `.isPersonalVoice` voice trait.
- Apps request authorization via `AVSpeechSynthesizer.requestPersonalVoiceAuthorization(_:)`.
- Intended primarily for AAC (Augmentative and Alternative Communication) apps.
- Works with Live Speech (type-to-speak feature) on iOS, iPadOS, macOS, and watchOS.

## APIs & Frameworks
- `AVFoundation` — speech synthesis framework
- `AVSpeechSynthesizer` — core synthesis object
- `AVSpeechUtterance` — utterance object
- `AVSpeechUtterance(ssmlRepresentation:)` **[NEW]** — initializer accepting SSML string
- `AVSpeechSynthesisVoice` — voice descriptor
- `AVSpeechSynthesisVoice.speechVoices()` — returns all available system voices including Personal Voices
- `AVSpeechSynthesisVoice.VoiceTrait.isPersonalVoice` **[NEW]** — trait flag identifying Personal Voices
- `AVSpeechSynthesizer.requestPersonalVoiceAuthorization(_:)` **[NEW]** — requests permission to use Personal Voice
- `AVSpeechSynthesizer.availableVoicesDidChangeNotification` **[NEW]** — notification posted when system voice list changes
- `AVSpeechSynthesisProviderAudioUnit` **[NEW]** — base class for custom speech synthesizer audio unit extensions
- `AVSpeechSynthesisProviderVoice` **[NEW]** — describes a voice provided by a third-party synthesizer
- `AVSpeechSynthesisProviderVoice.updateSpeechVoices()` **[NEW]** — static method to notify system of voice list changes
- `AVSpeechSynthesisProviderRequest` **[NEW]** — encapsulates SSML and voice for a synthesis request; properties: `ssmlRepresentation`, `voice`
- `AVSpeechSynthesisProviderAudioUnit.speechVoices` **[NEW]** — getter returning available voices from the extension
- `AVSpeechSynthesisProviderAudioUnit.synthesizeSpeechRequest(_:)` **[NEW]** — called when synthesis should begin
- `AVSpeechSynthesisProviderAudioUnit.cancelSpeechRequest()` **[NEW]** — called to cancel current synthesis
- `AUAudioUnit.internalRenderBlock` — render block filled with synthesized audio frames
- `AUInternalRenderBlock` — block type for the audio unit render callback
- `offlineUnitRenderAction_Complete` — flag set when all audio frames have been rendered
- `AudioUnit` framework — underlying audio unit architecture
- App Groups / `UserDefaults(suiteName:)` — shared storage between host app and extension for voice list
- SSML `<break>` tag — inserts pauses into synthesized speech
- SSML `<prosody rate="">` tag — adjusts speech rate percentage

## Code Highlights

Creating an SSML utterance with a pause and rate change:
```swift
let ssml = """
    <speak>
        Hello
        <break time="1s" />
        <prosody rate="200%">nice to meet you!</prosody>
    </speak>
"""
guard let ssmlUtterance = AVSpeechUtterance(ssmlRepresentation: ssml) else { return }
self.synthesizer.speak(ssmlUtterance)
```

Implementing a speech synthesis provider audio unit:
```swift
public class WWDCSynthAudioUnit: AVSpeechSynthesisProviderAudioUnit {
    public override var speechVoices: [AVSpeechSynthesisProviderVoice] {
        get {
            let voices = groupDefaults.value(forKey: "voices") as? [String: String] ?? [:]
            return voices.map { key, value in
                AVSpeechSynthesisProviderVoice(name: value, identifier: key,
                    primaryLanguages: ["en-US"], supportedLanguages: ["en-US"])
            }
        }
    }

    public override func synthesizeSpeechRequest(speechRequest: AVSpeechSynthesisProviderRequest) {
        currentBuffer = getAudioBuffer(for: speechRequest.voice, with: speechRequest.ssmlRepresentation)
        framePosition = 0
    }

    public override func cancelSpeechRequest() { currentBuffer = nil }
}
```

Requesting and using Personal Voice:
```swift
func fetchPersonalVoices() async {
    AVSpeechSynthesizer.requestPersonalVoiceAuthorization { status in
        if status == .authorized {
            personalVoices = AVSpeechSynthesisVoice.speechVoices()
                .filter { $0.voiceTraits.contains(.isPersonalVoice) }
        }
    }
}
```

## Takeaways
- Third-party speech synthesizers can now be registered system-wide via the Speech Synthesis Provider Audio Unit Extension, appearing in VoiceOver and any app that uses `AVSpeechSynthesizer`.
- SSML is the standard input format for custom synthesis extensions; use `AVSpeechUtterance(ssmlRepresentation:)` to construct utterances.
- Personal Voice is on-device, privacy-preserving, and intended for AAC apps — request authorization via `requestPersonalVoiceAuthorization` and filter voices by the `.isPersonalVoice` trait.
- `AVSpeechSynthesisProviderVoice.updateSpeechVoices()` is the key signal to keep system voice lists in sync after purchases or changes.

---
_Source: WWDC23 Session 10033 page (abstract, chapter summaries, code samples, and resource links)._
