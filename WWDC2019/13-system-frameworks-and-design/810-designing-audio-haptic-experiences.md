# Designing Audio-Haptic Experiences
**WWDC19 · Session 810** · [Watch](https://developer.apple.com/videos/play/wwdc2019/810/)

_Platforms:_ iOS 13, watchOS 6

## Overview
This design-focused session introduces the principles and craft of combining sound and haptics into unified audio-haptic experiences using the Taptic Engine and the new Core Haptics framework (iOS 13). Presented by Apple designers from the interaction and sound design teams, it covers the fundamental building blocks of Core Haptics (transient and continuous events, intensity, sharpness), three guiding design principles (causality, harmony, utility), and practical composition techniques learned from designing Apple's own system experiences — Apple Pay confirmation, Apple Watch crown rotation, Messages full-screen effects, and watchOS notification patterns.

The session is primarily a design guide rather than an API tutorial, emphasizing that audio, haptics, and visuals must be authored together as a single unified composition, with each element reinforcing the others through timing, energy level, and character.

## Key Topics

- **Core Haptics primitives** — Two event types: **Transient** (single-cycle impact, like a tap or strike) and **Continuous** (extended over a defined duration, like a rumble or buzz). Two design parameters each: **Haptic Intensity** (amplitude) and **Haptic Sharpness** (low = round/soft/organic; high = precise/mechanical/crisp). **[NEW in iOS 13]**
- **Three guiding principles:**
  - **Causality** — Feedback must make the cause obvious. Ask what physical material and dynamics would produce this sound and feel; design audio-haptic response to match. Example: Apple Pay confirmation — two simple taps paired with a chime, synchronized to the check-mark animation.
  - **Harmony** — Sound, haptics, and visuals must tell a consistent story. Synchronization is where the "illusion" of physicality is created; even small latency destroys it. Match energy level, pace, and character across all three senses. Example: Apple Watch crown — sharp transient haptics + quiet mechanical sound + animation snap.
  - **Utility** — Only add audio-haptic feedback where it provides clear value. Don't add feedback to every interaction; doing so overwhelms users and diminishes the impact of important moments. Start conservative and ask whether each addition genuinely helps.
- **Composition techniques:**
  - Match sound attack character to haptic type (sharp attack + transient; smooth ramp + continuous).
  - Invert expected pairing for creative effect — e.g., Apple Watch alarm uses a haptic ramp-up followed by the chime as a "response," creating anticipation.
  - "Ghost haptic" priming — the first event in a rapid sequence may not be consciously felt, acting as a skin primer that makes subsequent events feel stronger. Used in watchOS third-party notifications.
  - Creating contrast between similar experiences — doubling the haptic for one direction (left vs. right navigation cues on watchOS) while keeping audio similar makes the distinction unmistakably clear.
- **Cross-discipline collaboration** — Best results come from animators, sound designers, and interaction designers working together from the start, not adding haptics as an afterthought.

## APIs & Frameworks

### Core Haptics **[NEW]**
- `CHHapticEngine` — the primary entry point; manages the Taptic Engine connection
- `CHHapticEvent` — describes a single haptic event
  - `CHHapticEvent.EventType.hapticTransient` — short, impactful feel
  - `CHHapticEvent.EventType.hapticContinuous` — sustained duration feel
  - `CHHapticEvent.EventType.audioCustom` — custom audio synchronized with haptics
  - `CHHapticEvent.EventType.audioContinuous` — sustained audio event
- `CHHapticEventParameter` — per-event parameters:
  - `.hapticIntensity` — 0.0 to 1.0; amplitude
  - `.hapticSharpness` — 0.0 (soft/organic) to 1.0 (crisp/mechanical)
  - `.attackTime`, `.decayTime`, `.releaseTime`, `.sustained` — envelope controls
- `CHHapticDynamicParameter` — real-time parameter modulation during playback
  - `.hapticIntensityControl`, `.hapticSharpnessControl`, `.audioVolumeControl`
- `CHHapticPattern` — a collection of `CHHapticEvent` objects forming a complete pattern
- `CHHapticPatternPlayer` — plays a `CHHapticPattern`; can be started, stopped, and updated in real time
- `CHHapticAdvancedPatternPlayer` — adds looping and completion handler support
- `CHHapticEngine.capabilitiesForHardware()` — check if Core Haptics is available on the device
- AHAP file format (Apple Haptic and Audio Pattern) — JSON-based file format for authoring and sharing haptic patterns

### System Haptics (pre-Core Haptics, still applicable)
- `UIImpactFeedbackGenerator` — for impact-style transient feedback
- `UISelectionFeedbackGenerator` — for selection changes
- `UINotificationFeedbackGenerator` — for success/warning/error notifications

## Code Highlights

Basic transient haptic with Core Haptics:

```swift
import CoreHaptics

guard CHHapticEngine.capabilitiesForHardware().supportsHaptics else { return }

let engine = try CHHapticEngine()
try engine.start()

let intensity = CHHapticEventParameter(parameterID: .hapticIntensity, value: 1.0)
let sharpness = CHHapticEventParameter(parameterID: .hapticSharpness, value: 0.8)

let event = CHHapticEvent(
    eventType: .hapticTransient,
    parameters: [intensity, sharpness],
    relativeTime: 0)

let pattern = try CHHapticPattern(events: [event], parameters: [])
let player = try engine.makePlayer(with: pattern)
try player.start(atTime: CHHapticTimeImmediate)
```

Synchronized audio + haptic pattern (loading from AHAP file):

```swift
let url = Bundle.main.url(forResource: "Confirmation", withExtension: "ahap")!
try engine.playPattern(from: url)
```

## Takeaways

- Design audio, haptics, and visuals as one unified composition from the start — retrofitting any one element after the others are finalized produces incoherent experiences.
- Tight synchronization (< a few milliseconds) between the haptic event, the sound onset, and the visual animation frame is what creates the convincing illusion of physicality; any perceptible lag breaks it.
- Haptic sharpness is the most distinctive creative lever in Core Haptics — low sharpness reads as organic/natural, high sharpness as mechanical/precise; choose based on the character of the interaction.
- Restraint is a design virtue: a single well-placed haptic moment at a critical interaction point has far more impact than haptic feedback on every tap and swipe.

---
_Source: WWDC19 Session 810 page (abstract, full transcript, and resource links)._
