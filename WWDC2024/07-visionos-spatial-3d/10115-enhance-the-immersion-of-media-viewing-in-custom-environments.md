# Enhance the Immersion of Media Viewing in Custom Environments
**WWDC24 · Session 10115** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10115/)

_Platforms:_ visionOS 2

## Overview
This session shows how to build a cinema-quality media viewing experience on Apple Vision Pro by combining `ImmersiveSpace`, `AVKit`, and new Reality Composer Pro components. The foundational pattern is `AVPlayerViewController` running fullscreen in an immersive space with a custom **docking region** that positions the video screen precisely within a 3D environment.

New features in visionOS 2 include media reflections (specular and diffuse) on environment surfaces, `Virtual Environment Probe` for realistic lighting, `Reverb` components for acoustic immersion, `immersiveContentBrightness` and `surroundingsEffect` for hand-lighting blend, an **Immersive Environment Picker** for discoverability, and **SharePlay** environment synchronization via `AVGroupExperienceCoordinator`.

## Key Topics

### Immersive Playback Foundation
`ImmersiveSpace` + `AVPlayerViewController` in fullscreen is the recommended stack. The docking behavior places the video screen in a fixed location; apps customize that location with a **Docking Region Component** in Reality Composer Pro's Video Dock preset. The `width` property of the docking region defines a 2.4:1 bounding box; larger videos scale to fit.

### Custom Docking Region
In Reality Composer Pro, add the Video Dock preset, select the Player entity, and adjust the Docking Region Component's position and width. On device, the coordinate origin is at the floor below the viewer.

### Media Reflections
Two ShaderGraph nodes in Reality Composer Pro: `Reflection_Specular` (direct mirror-like reflection, good for glossy surfaces) and `Reflection_Diffuse` (soft falloff, good for floors). RealityKit computes reflections automatically when a docking region entity is present in the scene.

### Grounding the Experience
- **Virtual Environment Probe**: describes illumination for a virtual location, blending virtual and real lighting. Supports two pre-generated `Environment Resources` with a blend factor for dark/light scene variants.
- **Brightness and Tint** (`immersiveContentBrightness`, `surroundingsEffect`): softens the edge between passthrough and virtual environment, tinting the viewer's hands to match.
- **Reverb Component**: new in visionOS 2, applies naturalistic reverb presets (medium room, outdoor, very large room) to spatial and system sounds in an immersive space.

### Immersive Environment Picker
`.immersiveEnvironmentPicker { ... }` modifier on a view adds custom environments to the system picker list alongside system environments. Each entry needs a title, image, and the `ImmersiveSpace` ID to open.

### SharePlay
Pass the `GroupSession` to both `AVPlaybackCoordinator` (for media sync) and the new `AVGroupExperienceCoordinator` on `AVPlayerViewController` (for environment sync).

## APIs & Frameworks

**AVKit**
- `AVPlayerViewController` — fullscreen docking participant (existing, highlighted)
- `AVPlayerViewController.groupExperienceCoordinator` **[NEW]**
  - `.coordinateWithSession(_:)` **[NEW]**
- `AVPlaybackCoordinator.coordinateWithSession(_:)` (existing)
- `AVGroupExperienceCoordinator` **[NEW]**

**SwiftUI / visionOS**
- `ImmersiveSpace` scene type (existing)
- `.immersiveEnvironmentPicker { ... }` modifier **[NEW]**
- `immersiveContentBrightness` **[NEW]**
- `surroundingsEffect` / `SurroundingsEffect` **[NEW / highlighted]**

**Reality Composer Pro components**
- **Docking Region Component** **[NEW]** — `width`, position/rotation
- **Virtual Environment Probe** **[NEW]** — blend factor between two `Environment Resource` assets
- **Reverb Component** **[NEW]** — reverb presets: `.mediumRoom`, `.outdoor`, `.veryLargeRoom`, etc.
- **Reflection_Specular** ShaderGraph node **[NEW]**
- **Reflection_Diffuse** ShaderGraph node **[NEW]**

**GroupActivities**
- `GroupSession` (existing)

## Code Highlights

```swift
// Add environments to the Immersive Environment Picker
WindowGroup {
    ContentView()
        .immersiveEnvironmentPicker {
            ForEach(viewModel.environmentItems) { item in
                Button(item.title, image: item.thumbnail) {
                    Task { await openImmersiveSpace(id: item.id) }
                }
            }
        }
}

// SharePlay: sync both media and environment
for await session in MyActivity.sessions() {
    playerViewController.player?.playbackCoordinator.coordinateWithSession(session)
    playerViewController.groupExperienceCoordinator.coordinateWithSession(session)
}
```

## Takeaways
- Use the Docking Region Component in Reality Composer Pro to control exactly where the video screen appears in your custom environment — test placement on-device, not just on a monitor.
- Add `Reflection_Diffuse` or `Reflection_Specular` nodes to floor/wall materials to create the sense that media is really playing in the space.
- Apply the Reverb Component with a preset that matches your environment's size and surface materials, even if you have no custom spatial audio, so system sounds sound natural.
- Call `playerViewController.groupExperienceCoordinator.coordinateWithSession(session)` alongside `playbackCoordinator` to sync environment state in SharePlay.

---
_Source: WWDC24 Session 10115 page (abstract, chapter summaries, code samples, and resource links)._
