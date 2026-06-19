# Introducing Core Haptics
**WWDC19 · Session 520** · [Watch](https://developer.apple.com/videos/play/wwdc2019/520/)

_Platforms:_ iOS 13 (iPhone 8 and later with Apple Taptic Engine)

## Overview
Core Haptics is a new event-based audio and haptic rendering API (synthesizer) for iPhone introduced in iOS 13. It provides precise, low-latency control over the Apple Taptic Engine, enabling developers to design fully customized haptic patterns synchronized with audio — something previously impossible without specialized equipment. Core Haptics does not replace `UIFeedbackGenerator`; instead, it targets use cases where developers want to be their own haptic and sound designers, with real-time modulation and tight synchronization to animations, physics events, or audio.

The framework works on all iPhones from iPhone 8 onward (hundreds of millions of devices) using a single API and a single file format. Apple guarantees consistent feel across all supported hardware, so developers can prototype and release using a single device. The Taptic Engine across these devices offers a wide, expressive range with unmatched precision and subtlety.

A key design insight is the audio-haptic duality: synchronized haptic and audio feedback is integral to Apple's own experiences (haptic home button, haptic crown, UIDatePicker scroll wheels) and elevates app immersion in games and AR apps. Core Haptics enables developers to achieve this same pairing.

## Key Topics

**Event Types**
Core Haptics is structured around `CHHapticEvent` objects, each with a type, a relative timestamp, an optional duration, and optional parameters:
- `HapticTransient` — instantaneous impact (like a gavel strike); ideal for collisions, taps, selections.
- `HapticContinuous` — sustained haptic texture with richer modulation controls; ideal for backgrounds, tension build-up.
- `AudioContinuous` — sustained synthesized audio, synchronized with haptics.
- `AudioCustom` — plays a developer-provided audio file in sync with haptics.

**Event Parameters**
- `HapticIntensity` (0–1): strength of haptic output; analogous to audio volume.
- `HapticSharpness` (0–1): perceptual quality from round/organic (0) to crisp/precise (1). No physical analog; unique to haptics design.
- `AudioVolume`, `AudioPitch`, `AudioPan`: audio-side controls.
- `AttackTime`, `DecayTime`, `Sustained`: envelope controls for continuous events.

**Basic Playback Recipe**
1. Create `CHHapticPattern` from an array of events (programmatically or from an AHAP file).
2. Create `CHHapticEngine` — initialize as early as possible.
3. Create `CHHapticPatternPlayer` from the engine and pattern.
4. Start the engine.
5. Start the player in immediate mode (`CHHapticTimeImmediate`) or scheduled mode (absolute timestamp for sync with other systems).
6. Optionally, stop the engine when no longer needed.

**Dynamic Parameters**
Dynamic parameters adjust the value of event parameters for all active and upcoming events during playback without rebuilding the pattern. They take effect at a specified timestamp and can be sent to the player in real time, enabling a single pattern to produce infinite haptic variations in response to app state.

**Apple Haptic Audio Pattern (AHAP) File Format**
AHAP is a JSON-based specification for describing Core Haptics patterns. It separates content from application code, enables sharing and editing of patterns outside the app, and can be read/written by any JSON library including Swift's `Codable`. The file contains a version string, an array of event dictionaries (time, type, duration, event parameters), optional dynamic parameter arrays, and optional parameter curves.

**Lifecycle and Best Practices**
- The `CHHapticEngine.stoppedHandler` property handles unexpected stops (audio session interruptions, app suspension).
- Keep the engine running for the lifetime of any screen with haptic interaction; restart if needed.
- For fire-and-forget patterns, the player continues playing until complete without needing to retain the player object.
- `CHHapticTimeImmediate` requests minimum-latency immediate playback.

## APIs & Frameworks

**Core Haptics** **[NEW]**

Content classes:
- `CHHapticEvent` — single haptic/audio event **[NEW]**
  - `CHHapticEvent.EventType` enum: `.hapticTransient`, `.hapticContinuous`, `.audioContinuous`, `.audioCustom` **[NEW]**
  - `CHHapticEvent.init(eventType:parameters:relativeTime:duration:)` **[NEW]**
- `CHHapticEventParameter` — parameter attached to an event **[NEW]**
  - `CHHapticEvent.ParameterID`: `.hapticIntensity`, `.hapticSharpness`, `.audioVolume`, `.audioPitch`, `.audioPan`, `.attackTime`, `.decayTime`, `.sustained` **[NEW]**
- `CHHapticDynamicParameter` — real-time parameter modulation **[NEW]**
  - `CHHapticDynamicParameter.ID` **[NEW]**
- `CHHapticParameterCurve` — time-varying parameter curves **[NEW]**
- `CHHapticPattern` — container for events and dynamic parameters **[NEW]**
  - `CHHapticPattern.init(events:parameters:)` **[NEW]**
  - `CHHapticPattern.init(dictionary:)` — load from `NSDictionary` / AHAP **[NEW]**

Engine and playback:
- `CHHapticEngine` — controls hardware; singleton-like lifecycle **[NEW]**
  - `CHHapticEngine.init(audioSession:)` **[NEW]**
  - `CHHapticEngine.start(completionHandler:)` **[NEW]**
  - `CHHapticEngine.stop(completionHandler:)` **[NEW]**
  - `CHHapticEngine.stoppedHandler: ((CHHapticEngine.StoppedReason) -> Void)?` **[NEW]**
  - `CHHapticEngine.makePlayer(with:) -> CHHapticPatternPlayer` **[NEW]**
  - `CHHapticEngine.makeAdvancedPlayer(with:) -> CHHapticAdvancedPatternPlayer` **[NEW]**
- `CHHapticPatternPlayer` protocol **[NEW]**
  - `start(atTime:)` — `CHHapticTimeImmediate` for immediate, or absolute timestamp **[NEW]**
  - `stop(atTime:)` **[NEW]**
  - `sendParameters(_:atTime:)` — send dynamic parameters during playback **[NEW]**
- `CHHapticAdvancedPatternPlayer` — extended player with looping and callbacks **[NEW]**
- `CHHapticTimeImmediate` — constant for immediate playback **[NEW]**

Supporting types:
- `CHHapticEngine.StoppedReason` enum **[NEW]**
- `CHHapticError` **[NEW]**

**Companion APIs** (not new, but used alongside Core Haptics)
- `UIFeedbackGenerator` / `UIImpactFeedbackGenerator` / `UISelectionFeedbackGenerator` / `UINotificationFeedbackGenerator` — still preferred for standard UIKit controls
- `AVAudioEngine` — for advanced audio synchronization with Core Haptics
- `CoreAnimation` — for animation synchronization via scheduled timestamps

## Code Highlights

Setting up the haptic engine:
```swift
import CoreHaptics

var hapticEngine: CHHapticEngine!

func setupHaptics() {
    do {
        hapticEngine = try CHHapticEngine()
        hapticEngine.stoppedHandler = { reason in
            // Handle unexpected stop (interruption, suspension)
        }
        try hapticEngine.start()
    } catch {
        print("Engine creation failed: \(error)")
    }
}
```

Creating and playing a pattern in response to a collision:
```swift
func playCollisionHaptic(velocity: Float) throws {
    let intensity = CHHapticEventParameter(parameterID: .hapticIntensity, value: velocity)
    let sharpness = CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.8)
    
    let hapticEvent = CHHapticEvent(eventType: .hapticTransient,
                                    parameters: [intensity, sharpness],
                                    relativeTime: 0)
    
    let audioEvent = CHHapticEvent(eventType: .audioContinuous,
                                   parameters: [
                                       CHHapticEventParameter(parameterID: .audioVolume, value: velocity),
                                       CHHapticEventParameter(parameterID: .decayTime, value: 0.3),
                                       CHHapticEventParameter(parameterID: .sustained, value: 0)
                                   ],
                                   relativeTime: 0,
                                   duration: 0.5)
    
    let pattern = try CHHapticPattern(events: [hapticEvent, audioEvent], parameters: [])
    let player = try hapticEngine.makePlayer(with: pattern)
    try player.start(atTime: CHHapticTimeImmediate)
}
```

Sending a dynamic parameter during playback to reduce intensity:
```swift
let reduction = CHHapticDynamicParameter(parameterID: .hapticIntensityControl,
                                          value: 0.3, relativeTime: 0.5)
try player.sendParameters([reduction], atTime: CHHapticTimeImmediate)
```

Loading from an AHAP file:
```swift
if let url = Bundle.main.url(forResource: "Haptic", withExtension: "ahap") {
    try hapticEngine.playPattern(from: url)
}
```

## Takeaways
- Core Haptics gives developers direct access to the Apple Taptic Engine for the first time, enabling synchronized audio-haptic experiences previously exclusive to Apple's own UI.
- Two key parameters — `HapticIntensity` and `HapticSharpness` — cover most use cases; sharpness has no physical analog and must be learned through experimentation with the Haptic Palette sample.
- Dynamic parameters enable a single pre-authored pattern to adapt in real time to user input or physics state, avoiding pattern recreation per event.
- The AHAP JSON file format is the canonical way to author, share, and version haptic content; it maps directly to the Swift `Codable` framework for easy serialization.

---
_Source: WWDC19 Session 520 page (abstract, chapter summaries, code samples, and resource links)._
