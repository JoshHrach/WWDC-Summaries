# Enhance Your Spatial Computing App with RealityKit Audio
**WWDC24 · Session 111801** · [Watch](https://developer.apple.com/videos/play/wwdc2024/111801/)

_Platforms:_ visionOS, visionOS 2

## Overview
RealityKit exposes a layered audio system purpose-built for spatial computing. This session surveys every layer — from simple file playback to real-time generated audio — and shows how to tune directionality, reverb, ambience, and mix groups. The goal is a consistent, convincing soundscape that anchors virtual objects in the user's physical space.

The session is structured around four progressive topics: standard audio playback via `Entity.playAudio`, spatial shaping with `SpatialAudioComponent`, real-time synthesis via `AudioGeneratorController`, and environmental mix management with `AudioMixGroupsComponent`.

## Key Topics

### Entity Playback
`Entity.playAudio(_:)` loads an audio resource and returns an `AudioPlaybackController`. The controller exposes `play()`, `pause()`, `stop()`, `fade(to:duration:)`, and `completionHandler`. Audio files can be bundled or loaded from disk as `AudioFileResource`. Multi-file resources can be packaged as `AudioFileGroupResource` for random or sequential selection.

### Spatial Audio Component
Adding `SpatialAudioComponent` to an entity makes its sound position-track in 3D. Key properties:
- `directivity: .beam(focus:)` — narrows how much sound radiates backward; values near 1.0 create a tight forward cone.
- `gain: Double` — overall level in decibels.
- `distanceAttenuation` — controls how quickly volume falls off with distance (linear, inverse, logarithmic models).
- `reverbLevel: Double` — how much of the reverb (from `ReverbComponent`) is blended in.

### Real-Time Audio Generation
`AudioGeneratorController` allows apps to push synthesized audio frames directly into RealityKit's spatial mixer. Attach it to an entity in the same way as a playback controller. The controller's callback fires on a dedicated high-priority thread; audio data is provided as `AVAudioPCMBuffer` frames. This enables procedural sound effects (engine rumbles, wind, music procedurally driven by game state) that track entity position in 3D.

### Reverb Component
`ReverbComponent` attaches to a parent entity and applies a room impulse response to child entities' audio. New in visionOS 2: named presets via `ReverbComponent(reverb: .preset(.veryLargeRoom))`. Available presets include `.smallRoom`, `.mediumRoom`, `.largeRoom`, `.veryLargeRoom`, `.outdoor`, and several others. Setting a reverb component at the root of a scene provides a default room response; child entities can override or mix.

### Ambient and Mix Groups
`AmbientAudioComponent` plays non-positional audio (background music, atmosphere) attached to an entity. `AudioMixGroupsComponent` establishes named mix buses. Assign audio sources to a group by setting their `mixGroup` identifier. Volume, reverb send, and pitch can be driven per-group from code — enabling pause/mute/duck patterns without managing individual playback controllers.

## APIs & Frameworks

**RealityKit**
- `Entity.playAudio(_:) -> AudioPlaybackController` (existing, highlighted)
- `AudioPlaybackController` — `.play()`, `.pause()`, `.stop()`, `.fade(to:duration:)`, `.completionHandler` (existing)
- `AudioFileResource` (existing)
- `AudioFileGroupResource` **[NEW]** — bundles multiple audio files for random/sequential selection
- `SpatialAudioComponent` **[NEW]**
  - `.directivity: AudioComponent.Directivity` — `.beam(focus: Double)` **[NEW]**
  - `.gain: Double`
  - `.distanceAttenuation: AudioComponent.DistanceAttenuation`
  - `.reverbLevel: Double`
- `AudioGeneratorController` **[NEW]** — real-time PCM push to spatial mixer
  - Callback: `(AVAudioPCMBuffer) -> Void` on high-priority audio thread
- `ReverbComponent` **[NEW]**
  - `init(reverb: AudioComponent.Reverb)` **[NEW]**
  - `AudioComponent.Reverb.preset(_:)` **[NEW]** — `.smallRoom`, `.mediumRoom`, `.largeRoom`, `.veryLargeRoom`, `.outdoor`, etc.
- `AmbientAudioComponent` **[NEW]** — non-positional ambient sound tied to entity
- `AudioMixGroupsComponent` **[NEW]**
  - `.AudioMixGroup` — named bus with volume/reverb/pitch control **[NEW]**

## Code Highlights

```swift
// Spatial audio with beam directivity
var spatialAudio = SpatialAudioComponent()
spatialAudio.directivity = .beam(focus: 0.75)
entity.components.set(spatialAudio)

// Reverb for a room-scale scene
let reverb = ReverbComponent(reverb: .preset(.veryLargeRoom))
sceneRoot.components.set(reverb)

// Real-time audio generation
let generator = AudioGeneratorController(format: format)
generator.renderBlock = { buffer in
    // fill buffer with synthesized samples
}
entity.components.set(generator)

// Mix group volume control
mixGroupsComponent["music"].gain = -6.0  // duck music by 6 dB
```

## Takeaways
- Attach `SpatialAudioComponent` to every entity that emits sound — even subtle tuning of `directivity` and `distanceAttenuation` makes virtual objects feel grounded in space.
- Use `ReverbComponent` presets to match your scene's apparent room size; a mismatch (e.g., outdoor reverb in a small virtual room) breaks immersion immediately.
- Reach for `AudioGeneratorController` only for procedural or real-time-driven effects — file-based playback is simpler and more memory-efficient for static sounds.
- Group background music and ambient effects under `AudioMixGroupsComponent` mix groups so you can duck or mute them as a unit without tracking individual controllers.

---
_Source: WWDC24 Session 111801 page (abstract, chapter summaries, code samples, and resource links)._
