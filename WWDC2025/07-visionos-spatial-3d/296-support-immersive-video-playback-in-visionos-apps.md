# Support Immersive Video Playback in visionOS Apps
**WWDC25 · Session 296** · [Watch](https://developer.apple.com/videos/play/wwdc2025/296/)

_Platforms:_ visionOS 26

## Overview
This session covers how to integrate immersive video playback into visionOS apps. It explains the different immersive video formats Apple Vision Pro supports (180°, 360°, stereoscopic MV-HEVC), how to present them using AVFoundation and RealityKit, how to customize the playback experience (environment dimming, passthrough blending), and best practices for UI during immersive playback.

## Key Topics

### Immersive Video Formats
Apple Vision Pro supports several spatial video and immersive video formats:
- **Stereoscopic MV-HEVC** — the format used for Apple Immersive Video (180° content); encoded as two eye views in a single container
- **180° and 360° equirectangular video** — monoscopic or stereoscopic spherical video
- Standard 2D and spatial video formats for in-window playback

### AVFoundation Integration
`AVPlayer` and `AVPlayerViewController` handle immersive video playback on visionOS automatically when the asset contains the appropriate track and format metadata. Apps can also use `AVPlayerLayer` within a `RealityKit` composition for more control over playback integration with 3D scenes.

### Immersive Space for Video
Immersive video is presented within a visionOS Immersive Space. The session explains how to open an `ImmersiveSpace` scene scoped to the video playback context, set environment dimming levels, and manage the transition in/out of immersive mode in a way that feels natural to users.

### Environment Dimming and Passthrough Blending
Apps can configure how much real-world passthrough is visible during immersive video playback using the environment dimming API. Full immersion dims passthrough completely; progressive dimming allows partial mixed reality viewing alongside video content.

### Custom Playback UI
During immersive playback, standard system chrome is hidden. The session covers how to present a custom SwiftUI overlay (using `RealityView` attachments or `ImmersiveSpace` overlays) for playback controls, chapter navigation, and subtitles.

### Diegetic vs. Non-Diegetic UI
Best practices distinguish between UI placed within the 3D scene (diegetic, following head tracking) and UI that hovers in front of the user (non-diegetic). The session recommends minimal, glanceable controls placed in a predictable location.

## APIs & Frameworks

**AVFoundation**
- `AVPlayer` — plays immersive video assets (MV-HEVC, equirectangular) automatically on visionOS
- `AVPlayerViewController` — system playback UI with built-in immersive video support on visionOS **[UPDATED]**
- `AVPlayerLayer` — for embedding playback in RealityKit scenes

**RealityKit / SwiftUI (visionOS)**
- `ImmersiveSpace` — scene type for presenting immersive video environments **[UPDATED]**
- Environment dimming API **[NEW/UPDATED]** — control passthrough blend level during immersive video
- `RealityView` attachments — host SwiftUI playback controls within a 3D immersive space
- `VideoPlayerComponent` — RealityKit component for attaching video playback to an entity in a scene

**AVKit**
- `AVPlayerViewController` on visionOS — presents full-immersive playback experience with system UI chrome

## Code Highlights

```swift
// Open an immersive space for video playback
@Environment(\.openImmersiveSpace) var openImmersiveSpace

Button("Play Immersive Video") {
    Task {
        await openImmersiveSpace(id: "ImmersiveVideoPlayer")
    }
}
```

```swift
// RealityKit: attach video to an entity
var body: some View {
    RealityView { content in
        let videoEntity = Entity()
        videoEntity.components[VideoPlayerComponent.self] = VideoPlayerComponent(avPlayer: player)
        content.add(videoEntity)
    }
}
```

## Takeaways
- Use `AVPlayerViewController` on visionOS for the quickest path to immersive video playback — the system handles format detection, environment transitions, and basic UI automatically.
- Implement a custom `ImmersiveSpace` scene for branded or interactive playback experiences that need custom UI or RealityKit integration.
- Configure environment dimming progressively rather than jumping to full immersion to ease the transition and accommodate users who prefer passthrough.
- Place playback controls at a predictable diegetic position in the scene, using glanceable SwiftUI overlays attached via `RealityView`.

---
_Source: WWDC25 Session 296 page (abstract, chapter summaries, and resource links). Note: full transcript was not accessible; summary is based on available preview content and session abstract._
