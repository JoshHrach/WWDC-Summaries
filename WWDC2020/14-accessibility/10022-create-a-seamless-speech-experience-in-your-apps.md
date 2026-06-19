# Create a Seamless Speech Experience in Your Apps
**WWDC20 · Session 10022** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10022/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, watchOS 7, tvOS 14

## Overview
This session explores how to integrate speech synthesis into apps using `AVSpeechSynthesizer` in ways that complement, rather than conflict with, existing assistive technologies like VoiceOver. The talk helps developers understand when `AVSpeechSynthesizer` is the right tool, when UIAccessibility APIs are more appropriate, and how to configure speech for a range of use cases.

The session emphasizes that `AVSpeechSynthesizer` is best suited for apps where speech is a core, universal experience — like navigation apps or AAC (Augmentative Alternative Communication) tools — rather than as an accessibility substitute. When speech is needed only alongside a screen reader, UIAccessibility notification APIs provide a cleaner and more consistent approach.

Advanced topics include routing speech through active phone or FaceTime calls, opting speech out of the shared application audio session, and a newly introduced API that respects the speech settings of whatever assistive technology is currently running on the device.

## Key Topics

**When to Use AVSpeechSynthesizer vs. UIAccessibility**
AVSpeechSynthesizer is appropriate when all users benefit from speech (e.g., driving directions). For VoiceOver-only announcements, use `UIAccessibility.post(notification:argument:)` instead. Building a custom screen reader with `AVSpeechSynthesizer` is explicitly discouraged.

**Getting Started with AVSpeechSynthesizer**
The minimum setup involves creating an `AVSpeechSynthesizer`, constructing an `AVSpeechUtterance` with the desired string, and calling `speak(_:)`. The synthesizer should be retained until speech completes.

**prefersAssistiveTechnologySettings (New in iOS 14)**
A new property on `AVSpeechUtterance` that, when set to `true`, causes AVSpeechSynthesizer to adopt the voice, rate, and pitch of the currently running assistive technology. This creates a more seamless, expected experience for users of VoiceOver or other AT.

**Customizing Voice, Rate, Pitch, and Volume**
Developers can override default settings by assigning a voice (via language code or identifier), adjusting `rate`, `pitchMultiplier`, and `volume` directly on `AVSpeechUtterance`.

**Routing Speech Through Outgoing Calls**
The `mixToTelephonyUplink` property on `AVSpeechSynthesizer` routes synthesized speech through an active phone or FaceTime call — particularly useful for AAC apps enabling nonverbal communication over telephony.

**Managing Audio Session Behavior**
`usesApplicationAudioSession` (default `true`) controls whether speech shares the app's audio session. Setting it to `false` delegates session management to the system, enabling speech to automatically duck and mix with other audio.

## APIs & Frameworks

### AVFoundation / AVSpeechSynthesis
- `AVSpeechSynthesizer` — core speech synthesis object
- `AVSpeechSynthesizer.speak(_:)` — initiates speech for a given utterance
- `AVSpeechSynthesizer.mixToTelephonyUplink` — routes speech through active outgoing call
- `AVSpeechSynthesizer.usesApplicationAudioSession` — controls audio session ownership
- `AVSpeechUtterance` — represents a unit of speech with text and settings
- `AVSpeechUtterance.prefersAssistiveTechnologySettings` **[NEW]** — adopts AT speech settings (voice, rate, pitch)
- `AVSpeechUtterance.voice` — assigns a specific `AVSpeechSynthesisVoice`
- `AVSpeechUtterance.rate` — speaking rate (0.0–1.0; default 0.5)
- `AVSpeechUtterance.pitchMultiplier` — pitch (0.5–2.0; default 1.0)
- `AVSpeechUtterance.volume` — volume (0.0–1.0; default 1.0)
- `AVSpeechSynthesisVoice(language:)` — selects voice by BCP-47 language code
- `AVSpeechSynthesisVoice(identifier:)` — selects voice by identifier
- `AVSpeechSynthesisVoice.speechVoices()` — returns all installed voices
- `AVSpeechSynthesisVoiceIdentifierAlex` — constant for the Alex voice

### UIKit / UIAccessibility
- `UIAccessibility.post(notification:argument:)` — posts accessibility notification (e.g., `.announcement`) to delegate speech to VoiceOver

## Code Highlights

Post an announcement to the running assistive technology instead of using `AVSpeechSynthesizer` directly:
```swift
UIAccessibility.post(notification: .announcement, argument: "Hello World")
```

Minimal AVSpeechSynthesizer usage:
```swift
self.synthesizer = AVSpeechSynthesizer()
let utterance = AVSpeechUtterance(string: "Hello World")
self.synthesizer.speak(utterance)
```

Using the new `prefersAssistiveTechnologySettings` API:
```swift
let utterance = AVSpeechUtterance(string: "Hello World")
utterance.prefersAssistiveTechnologySettings = true
self.synthesizer.speak(utterance)
```

Routing speech through an outgoing call:
```swift
self.synthesizer = AVSpeechSynthesizer()
self.synthesizer.mixToTelephonyUplink = true
```

Opting out of the application audio session:
```swift
self.synthesizer = AVSpeechSynthesizer()
self.synthesizer.usesApplicationAudioSession = false
```

## Takeaways
- Use `AVSpeechSynthesizer` for universal speech features and AAC-style apps; use `UIAccessibility` APIs for VoiceOver-specific announcements.
- The new `prefersAssistiveTechnologySettings` property on `AVSpeechUtterance` seamlessly aligns synthesized speech with whatever AT the user has configured.
- `mixToTelephonyUplink` enables AAC apps to route synthesized speech through phone and FaceTime calls, expanding communication possibilities for nonverbal users.
- Setting `usesApplicationAudioSession = false` hands off audio session lifecycle management to the system, simplifying mixing and interruption handling.

---
_Source: WWDC20 Session 10022 page (abstract, chapter summaries, code samples, and resource links)._
