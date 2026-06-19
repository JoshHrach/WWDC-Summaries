# Practice Audio Haptic Design
**WWDC21 · Session 10278** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10278/)

_Platforms:_ iOS 15 (iPhone 8 or newer required for haptic playback)

## Overview
This session provides a hands-on walkthrough of designing rich multimodal experiences that combine animation, sound, and haptics using the Core Haptics framework. Using the HapticRicochet sample app — a physics-based ball game where a ball rolls around and bounces off the iPhone's edges — presenter Camille Moussette from the Apple Design team demonstrates how to craft haptic and audio feedback that feels harmonious and purposeful.

The session covers three key design principles: causality (making feedback's source obvious), harmony (ensuring senses reinforce each other), and utility (reserving haptics for meaningful moments). Developers learn to load and play `.ahap` (Apple Haptic Audio Pattern) files, combine haptic and audio assets from different files, and use `CHHapticAdvancedPatternPlayer` for looping textures.

By iterating on two moments — adding a shield to the ball and enabling a rolling texture — the session demonstrates a practical, iterative design workflow where haptic and audio tracks are refined independently and then harmonized for the best combined experience.

## Key Topics
- **Design Principles**: Causality, Harmony, and Utility as a framework for multimodal design decisions
- **Core Haptics Architecture**: Engine → Player → Pattern → Events hierarchy
- **Haptic Event Types**: Transient events (sharp, single-moment) vs. continuous events (sustained sensations)
- **AHAP File Format**: JSON-based Apple Haptic Audio Pattern files; viewing with QuickLook Visualizer on macOS 12
- **Combining Assets**: Mixing audio from one `.ahap` file with haptics from another via `EventType AudioCustom` and filename references
- **Looping Patterns**: Using `CHHapticAdvancedPatternPlayer` to loop a texture pattern infinitely with no seam
- **Dynamic Intensity**: Updating haptic intensity at runtime based on ball speed via `updateTexturePlayer`
- **Shield Transformation**: A continuous haptic that conveys gaining protection, paired with matching audio
- **Rolling Texture**: A looping haptic pattern representing a ball rolling over a surface; linked to a visual background texture (causality)

## APIs & Frameworks
- **`CoreHaptics`** framework
- **`CHHapticEngine`** — top-level object linking to the device's haptic actuator
- **`CHHapticPatternPlayer`** — standard player for start, stop, pause playback
- **`CHHapticAdvancedPatternPlayer`** **[NEW in use emphasis]** — advanced player adding looping (`loopEnabled`), pause/resume, and callbacks
- **`CHHapticPattern`** — a time-ordered collection of `CHHapticEvent` objects
- **`CHHapticEvent`** — building block for a single haptic/audio moment; types include `.hapticTransient`, `.hapticContinuous`, `.audioContinuous`, `.audioCustom`
- **`CHHapticEventParameter`** — parameters for events: `HapticIntensity`, `HapticSharpness`, `AudioVolume`
- **`CHHapticDynamicParameter`** — used to update playback parameters at runtime (e.g., intensity from ball speed)
- **AHAP file format** (`.ahap`) — JSON file describing haptic+audio patterns; supports `EventType: HapticTransient`, `HapticContinuous`, `AudioCustom`
- **`engine.makePlayer(with:)`** — creates a `CHHapticPatternPlayer` from a `CHHapticPattern`
- **`engine.makeAdvancedPlayer(with:)`** — creates a `CHHapticAdvancedPatternPlayer`
- **QuickLook Visualizer** for `.ahap` files on macOS 12 — press Space in Finder to preview patterns visually

## Code Highlights
```swift
// Initialize shield haptics
func initializeShieldHaptics() {
    let pattern = createPatternFromAHAP("ShieldTransient")!
    shieldPlayer = try? engine.makePlayer(with: pattern)
}

// Play shield transformation
func shield() {
    startPlayer(shieldPlayer)          // plays haptic + audio
    isAnimating = true
    sphereView.layer.add(shieldAnimation, forKey: "Width")
}

// Use CHHapticAdvancedPatternPlayer for looping texture
var texturePlayer: CHHapticAdvancedPatternPlayer?

func initializeTextureHaptics() {
    let pattern = createPatternFromAHAP("Texture")!
    texturePlayer = try? engine.makeAdvancedPlayer(with: pattern)
    texturePlayer?.loopEnabled = true
}
```

Mixing audio from one AHAP into another is done by editing the `EventType: AudioCustom` entry in the JSON to reference the preferred `.wav` filename.

## Takeaways
- Multimodal experiences feel magical when haptic, audio, and visual elements are causally linked, harmonious in quality, and reserved for meaningful moments.
- Swapping audio between `.ahap` files is as simple as changing the `AudioCustom` filename in JSON — no code changes needed.
- `CHHapticAdvancedPatternPlayer` with `loopEnabled = true` is the right choice for sustained ambient textures like rolling surfaces.
- Iterative refinement is key: prototype each sensory channel independently, then evaluate their combined effect on real hardware (simulator does not support haptics).

---
_Source: WWDC21 Session 10278 page (abstract, transcript, code samples, and resource links)._
