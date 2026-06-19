# Expanding the Sensory Experience with Core Haptics
**WWDC19 · Session 223** · [Watch](https://developer.apple.com/videos/play/wwdc2019/223/)

_Platforms:_ iOS 13 (iPhone 8 and later)

## Overview
Core Haptics is a new event-based audio and haptic rendering API introduced in iOS 13 that gives developers full access to the Taptic Engine in iPhone 8 and later. The session is split into two halves: a design primer from Apple's interaction and sound designers, followed by a technical deep-dive into the API with code examples.

The design half introduces three guiding principles — Causality, Harmony, and Utility — illustrated through real Apple product examples including Apple Pay confirmation, Apple Watch crown feedback, and Messages full-screen effects. The engineering half covers the class model, event types, parameters, playback recipe, dynamic parameter adjustment at runtime, and the new Apple Haptic Audio Pattern (AHAP) file format for storing and sharing haptic content.

Core Haptics is not a replacement for `UIFeedbackGenerator`; it is designed for cases where developers want full authorship over custom haptic patterns and need tight synchronization with audio and animation.

## Key Topics

**Design Principles**
- Causality: feedback must clearly relate to its cause; match material and dynamics of real-world interactions
- Harmony: sound, haptics, and visuals must be synchronized and share consistent qualities (energy, pace, material feel)
- Utility: add haptics only where they provide clear value; more is not better

**Core Haptics vs. UIFeedbackGenerator**
- `UIFeedbackGenerator` handles intent-based haptics (selection, impact, notification); Apple manages the vocabulary
- `CHHapticEngine` is for custom patterns where exact timing, intensity, and synchronization with other APIs matter

**Event Types and Parameters**
- Haptic Transient: momentary impact/tap feel
- Haptic Continuous: extended texture/rumble
- Audio Continuous: synthesized audio synchronized with haptics
- Audio Custom: custom waveform audio in sync with haptics
- Parameters: `HapticIntensity` (0–1 amplitude), `HapticSharpness` (0=round/organic, 1=crisp/precise), `AudioVolume`, `AudioPitch`, `AudioPan`, decay envelope

**Playback Recipe**
Create pattern → create engine → create player → start engine → start player (immediate or scheduled with absolute timestamp) → handle stop/completion callbacks

**Dynamic Parameters**
Adjust event parameters for all active and upcoming events at any timestamp during playback — enables real-time modulation in response to user input, physics, or game state

**AHAP File Format**
JSON-based Apple Haptic Audio Pattern format; nested key/value pairs for version, pattern (events + parameters), dynamic parameter curves; loadable via Swift `Codable`

## APIs & Frameworks

**CoreHaptics** **[NEW]**
- `CHHapticEngine` **[NEW]** — main engine class; manages audio/haptic hardware lifecycle
  - `init()` **[NEW]**
  - `start(completionHandler:)` **[NEW]**
  - `stop(completionHandler:)` **[NEW]**
  - `stoppedHandler: CHHapticEngine.StoppedHandler?` **[NEW]** — callback when engine stops unexpectedly
  - `makePlayer(with:) -> CHHapticPatternPlayer` **[NEW]**
  - `makeAdvancedPlayer(with:) -> CHHapticAdvancedPatternPlayer` **[NEW]**
- `CHHapticPattern` **[NEW]** — container for events and dynamic parameters
  - `init(events:parameters:)` **[NEW]**
- `CHHapticEvent` **[NEW]** — single haptic or audio event
  - `init(eventType:parameters:relativeTime:)` **[NEW]**
  - `init(eventType:parameters:relativeTime:duration:)` **[NEW]**
  - `CHHapticEvent.EventType` **[NEW]**:
    - `.hapticTransient` **[NEW]**
    - `.hapticContinuous` **[NEW]**
    - `.audioContinuous` **[NEW]**
    - `.audioCustom` **[NEW]**
- `CHHapticEventParameter` **[NEW]** — per-event parameter
  - `init(parameterID:value:)` **[NEW]**
  - `CHHapticEvent.ParameterID` **[NEW]**: `.hapticIntensity`, `.hapticSharpness`, `.audioVolume`, `.audioPitch`, `.audioPan`, `.decayTime`, `.sustained`
- `CHHapticDynamicParameter` **[NEW]** — runtime parameter adjustment
  - `init(parameterID:value:relativeTime:)` **[NEW]**
  - `CHHapticDynamicParameter.ID` **[NEW]**: `.hapticIntensityControl`, `.hapticSharpnessControl`, `.audioVolumeControl`, etc.
- `CHHapticParameterCurve` **[NEW]** — smooth interpolated parameter changes over time
- `CHHapticPatternPlayer` protocol **[NEW]**
  - `func start(atTime time: TimeInterval) throws` **[NEW]**
  - `func stop(atTime time: TimeInterval) throws` **[NEW]**
  - `func sendParameters(_:atTime:) throws` **[NEW]** — real-time dynamic parameter updates
- `CHHapticTimeImmediate` **[NEW]** — constant for immediate playback
- AHAP file format (`.ahap`) **[NEW]** — JSON schema for haptic pattern files; loadable with `CHHapticPattern(contentsOfFile:)`

**UIKit**
- `UIFeedbackGenerator` and subclasses (existing) — improved in iOS 13 but separate from Core Haptics

## Code Highlights

Basic setup and play:
```swift
import CoreHaptics

var engine: CHHapticEngine!

func setupHaptics() {
    engine = try! CHHapticEngine()
    engine.stoppedHandler = { reason in
        // handle unexpected stop
    }
    try! engine.start()
}

func playCollision(velocity: Float) {
    let intensity = CHHapticEventParameter(parameterID: .hapticIntensity, value: velocity)
    let sharpness = CHHapticEventParameter(parameterID: .hapticSharpness, value: 1.0)
    let hapticEvent = CHHapticEvent(eventType: .hapticTransient, parameters: [intensity, sharpness], relativeTime: 0)

    let volume = CHHapticEventParameter(parameterID: .audioVolume, value: velocity)
    let decay = CHHapticEventParameter(parameterID: .decayTime, value: 0.2)
    let sustained = CHHapticEventParameter(parameterID: .sustained, value: 0)
    let audioEvent = CHHapticEvent(eventType: .audioContinuous, parameters: [volume, decay, sustained], relativeTime: 0, duration: 0.5)

    let pattern = try! CHHapticPattern(events: [hapticEvent, audioEvent], parameters: [])
    let player = try! engine.makePlayer(with: pattern)
    try! player.start(atTime: CHHapticTimeImmediate)
}
```

Dynamic parameter adjustment at runtime:
```swift
let param = CHHapticDynamicParameter(parameterID: .hapticIntensityControl, value: 0.3, relativeTime: 0.5)
try! player.sendParameters([param], atTime: CHHapticTimeImmediate)
```

## Takeaways
- Use Core Haptics when you need custom haptic patterns with tight audio/animation synchronization; use `UIFeedbackGenerator` for standard UIKit interaction feedback.
- Follow the Causality, Harmony, and Utility design principles — don't add haptics just because the API exists.
- Use Dynamic Parameters and parameter curves for real-time modulation in games, ARKit apps, and physics-driven UIs.
- Store patterns as `.ahap` JSON files to separate content from code and enable easy iteration by designers.

---
_Source: WWDC19 Session 223 page (abstract, chapter summaries, code samples, and resource links)._
