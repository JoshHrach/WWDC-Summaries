# Create a Great Spatial Playback Experience
**WWDC23 · Session 10070** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10070/)

_Platforms:_ visionOS 1

## Overview
visionOS introduces a new dimension to video playback: a large floating screen with spatial audio that fills your space, dims the room for cinema-like ambiance, and can dock inside an immersive space for a fully customized environment. This session walks through the AVFoundation and AVKit APIs that power this experience, showing how apps built with existing iOS APIs can get most of these features for free, and when and how to use lower-level APIs for specific use cases.

The session is structured around four video display APIs—`AVPlayerViewController` (fullscreen), `AVPlayerViewController` (inline), `VideoPlayerComponent`, and `VideoMaterial`—with guidance on which to choose for different scenarios. The key message is that `AVPlayerViewController` fullscreen should be the default choice, as it delivers complete system integration (Dimming Effect, Digital Crown volume, docking into Immersive Spaces, thumbnail scrubbing, interstitials, captions, and contextual actions) at no cost.

## Key Topics

### Platform Architecture
AVFoundation is enhanced on visionOS to decode and render 3D video formats and to composite video seamlessly into the world using RealityKit. Audio responds to the spatial environment. AVKit's `AVPlayerViewController` uses RealityKit internally to deliver high-quality compositing and supports new platform-specific features.

### AVPlayerViewController Fullscreen (Recommended)
Getting started requires the same code as iOS/tvOS: create `AVPlayerViewController`, connect `AVPlayer`, set a `AVPlayerItem`, present to fill the window. Key behaviors:
- A large screen appears in front of the user; the room dims for ambiance.
- Screen stays fixed in space while the user moves; audio anchors to the screen.
- Controls appear on look + tap; disappear automatically or on second tap.
- Window bar allows repositioning; corner resize maintains aspect ratio.
- Digital Crown adjusts volume or opens an environment overlay.
- **Dimming Effect** toggle in the controls menu.

### Built-in Playback Controls
- Volume control (top right) / Digital Crown
- Play/pause, back/forward
- Timeline scrubber
- Options menu: playback speed, audio track selection, caption track selection, Dimming Effect

### Advanced UI Features
- **Thumbnail scrubbing** — automatic for HLS streams with I-frame Trick Play tracks (preferred width: 145px).
- **Interstitials** — configured via `AVPlayerInterstitialEventController` or HLS stream; timeline indicator shown automatically.
- **Contextual actions** — `AVPlayerViewController` API (e.g., "Skip intro", "Play next episode"); title and optional image.
- **Custom info view controllers** — metadata or related content panels.

### Immersive Space Integration
When an app opens an `ImmersiveSpace` while video is playing, `AVPlayerViewController` automatically docks the video screen at a fixed, optimal position and size within the space. Playback controls detach and move closer. The app defines the 3D environment content using RealityKit entities; the system handles docking.

### Inline Playback (AVPlayerViewController)
When `AVPlayerViewController` does not fill the window, it automatically uses inline mode. Uses `AVPlayerLayer` for compositing within the window—cannot display 3D video.

### VideoPlayerComponent (RealityKit)
`VideoPlayerComponent` connects an `AVPlayer` to a RealityKit `Entity`, rendering it in 3D space as part of the scene. Creates an aspect-ratio-correct mesh automatically, supports captions, supports 3D video formats, and benefits from RealityKit rendering optimizations. Preferred over `AVPlayerLayer` when embedding video in a 3D scene.

### VideoMaterial (Lower-Level)
`VideoMaterial` renders video on arbitrary custom geometry. No aspect ratio enforcement, no caption support. Use only when custom geometry control is required.

### API Comparison
| API | 3D Video | Captions | System Integration | Custom Geometry |
|---|---|---|---|---|
| `AVPlayerViewController` fullscreen | Yes | Yes | Full | No |
| `AVPlayerViewController` inline | No | Yes | Partial | No |
| `VideoPlayerComponent` | Yes | Yes | No docking | No |
| `VideoMaterial` | Yes | No | No | Yes |

## APIs & Frameworks

**AVKit (visionOS)**
- `AVPlayerViewController` — primary video playback view controller; now with RealityKit rendering and visionOS-specific controls **[UPDATED]**
- `AVPlayerViewController` — custom info view controllers property
- `AVPlayerViewController` — contextual actions API (same as other platforms)
- Dimming Effect — new control option in playback controls UI **[NEW]**
- Immersive Space docking — automatic when `ImmersiveSpace` is open **[NEW]**
- Inline mode — automatic when not filling window

**AVFoundation**
- `AVPlayer` — core media playback engine
- `AVPlayerItem` — represents media to play
- `AVPlayerLayer` — CALayer-based video rendering (not RealityKit; no 3D video)
- `AVPlayerInterstitialEventController` — programmatic interstitial configuration

**RealityKit (visionOS)**
- `VideoPlayerComponent` **[NEW]** — entity component connecting `AVPlayer` to a RealityKit scene entity; supports 3D video and captions
- `VideoMaterial` — material applying video to arbitrary mesh geometry

**SwiftUI (visionOS)**
- `ImmersiveSpace(id:)` — scene type; video docks automatically when open
- `UIViewControllerRepresentable` — wrap `AVPlayerViewController` for SwiftUI

**HLS / Media Format**
- I-frame Trick Play track — enables thumbnail scrubbing (145px width preferred)
- HLS interstitials — described in stream; automatically reflected in timeline
- 3D video formats — new formats supported by AVFoundation on visionOS

## Code Highlights

Minimal video playing app on visionOS:
```swift
import AVFoundation
import AVKit
import SwiftUI

struct PlayerView: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> AVPlayerViewController {
        let controller = AVPlayerViewController()
        let player = AVPlayer()
        controller.player = player
        let item = AVPlayerItem(url: contentURL)
        player.replaceCurrentItem(with: item)
        return controller
    }
    func updateUIViewController(_ uiViewController: AVPlayerViewController, context: Context) {}
}

@main
struct MoviePlayingApp: App {
    var body: some Scene {
        WindowGroup { PlayerView() }
    }
}
```

Adding an Immersive Space for docked playback:
```swift
@main
struct MoviePlayingApp: App {
    var body: some Scene {
        WindowGroup { PlayerView().onAppear { openImmersiveSpace(id: "cinema") } }
        ImmersiveSpace(id: "cinema") { RealityView { content in /* 3D environment */ } }
    }
}
```

## Takeaways
- `AVPlayerViewController` fullscreen is the default recommendation for visionOS; existing iOS/tvOS code works with minimal changes and unlocks the full spatial experience.
- `VideoPlayerComponent` is the right choice for embedding video within a 3D RealityKit scene—it outperforms `AVPlayerLayer` and supports 3D video formats.
- Immersive Space integration is automatic: when an `ImmersiveSpace` is open, the system docks the video screen at an optimal position with no extra app code.
- `AVPlayerLayer` and `VideoMaterial` are specialized tools for specific cases; `VideoMaterial` sacrifices captions and aspect ratio for arbitrary geometry control.

---
_Source: WWDC23 Session 10070 page (abstract, chapter summaries, and resource links)._
