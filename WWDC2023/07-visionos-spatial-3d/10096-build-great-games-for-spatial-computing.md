# Build Great Games for Spatial Computing
**WWDC23 · Session 10096** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10096/)

_Platforms:_ visionOS 1

## Overview
This is the game-developer roadmap session for visionOS. It surveys the full landscape of what's possible — game types, the platform's unique rendering/audio/input characteristics, and the available framework stack — then focuses on RealityKit as the primary recommended framework for building native spatial games. The session is strategic and conceptual, directing developers to deeper dive sessions for each technology area.

The central message: visionOS supports a spectrum of immersion from shared-space windowed games to fully immersive experiences, and developers choose where on that spectrum to position their game based on how much of the player's attention they want to capture.

## Key Topics

### Types of Spatial Games
- **Shared Space**: game lives alongside other apps and system UI; good for ambient or casual games (e.g., a chessboard on the real desk, a virtual pet on the floor). Player interacts via Look & Tap.
- **Full Space**: closes other windows/volumes; game is the focus, but passthrough keeps the player connected to the real world. Good for action games that interact with the environment.
- **Fully Immersive**: app takes over the entire view; passthrough disabled; player sees only the rendered environment.
- **Windowed 2D games**: run automatically as compatible iOS/iPadOS apps in a virtual window; no spatial API adoption needed. Players can resize and reposition the window freely.
- **2.5D games**: 2D games enhanced with depth layers (parallax), volumetric elements emerging from the plane (sparks, smoke), or custom hand gestures.

### Rendering
- In Shared Space: rendering is composited by the system alongside other apps, passthrough, and system UI. Apps use surface/geometry shaders via **MaterialX**; can't use custom Metal pipelines.
- RealityKit default: physically based rendering (PBR — roughness, specular, metallic) with **real-time sampled room lighting** automatically applied for realism.
- **Dynamic Foveation**: platform renderer automatically applies higher pixel density where the player's eyes are focused — no developer work required.
- System effects applied to all Shared Space content: depth mitigation (transparency behind real objects), near-field vignetting, breakthrough (nearby person breaks through), grounding shadows.
- Custom shaders: use MaterialX shader graphs (editable in Reality Composer Pro); assign custom IBLs for custom lighting effects.
- **Fully Immersive rendering**: use `CompositorServices` + ARKit for full custom Metal pipeline; renders for each eye separately; replaces passthrough completely.

### Audio
- visionOS uses **Spatial Audio** that automatically matches reverb and room acoustics to the player's real environment.
- Standard iOS audio APIs (AVAudioEngine): audio positioned relative to the app window.
- **RealityKit audio on entities**: attach audio to specific RealityKit entities to spatialize sound relative to objects in the scene.
- Custom head-tracked audio: use ARKit to get the player's head position for manual spatial audio calculations.

### Input
- **Look & Tap (system gestures)**: player looks at an element and taps fingers together to select; covers drag, magnify, and other standard gestures.
  - Requires `CollisionComponent` (collision shape) + `InputTargetComponent` (marks entity as interactable) on RealityKit entities.
- **Bluetooth controllers, trackpads, mice, keyboards**: all supported, same as iOS.
- **Hand tracking** (ARKit): access raw hand joint data for custom gestures (grabbing, pointing, etc.); requires user permission; hands must be camera-visible.
- **Scene understanding** (ARKit, Full Space only): virtual mesh of the room, plane detection (horizontal/vertical surfaces), surface material detection (carpet/wood); requires user permission.

### Framework Choices
| Use case | Recommended framework |
|----------|----------------------|
| 2D games, enhanced with spatial elements | SwiftUI, SpriteKit |
| 3D spatial games, volumes, immersive spaces | **RealityKit** |
| Porting existing Unity game | Unity (visionOS plug-in) |
| Custom engine, full Metal control | CompositorServices + ARKit + Metal |

### RealityKit for Games
- Entity component system (ECS) for custom behaviors and extensibility.
- Physics, animation, particles, audio.
- USD model loading and custom mesh support.
- MaterialX and IBL lighting for PBR and custom materials.
- **Attachments** — connect SwiftUI views directly to RealityKit entities (new feature).
- Scene primitives: `Window` (player-controlled size), `Volume` (developer-controlled fixed size — best for games), `Space` (shared or full), `Anchor` (attach to real-world surfaces or hands), `Portal` (cut a hole in reality showing a rendered world inside).
- `RealityView` — SwiftUI view that hosts RealityKit rendering and simulation.
- Reality Composer Pro — visual tool bundled with Xcode for assembling USD scenes, editing MaterialX shaders, previewing on device; automatically optimizes assets.

## APIs & Frameworks

### RealityKit **[NEW on visionOS]**
- `RealityView` — SwiftUI view wrapping RealityKit rendering **[NEW]**
- `Entity` — base ECS entity type
- `CollisionComponent` — adds collision shape to entity (required for gesture input)
- `InputTargetComponent` — marks entity as interactable with system gestures **[NEW]**
- `AnchorEntity` — anchors content to real-world surfaces, hands, or world origin
- `ModelEntity` — entity that renders a 3D model (USD or custom mesh)
- `AudioLibraryComponent` / `SpatialAudioComponent` — spatial audio on entities
- Attachments — SwiftUI views attached to RealityKit entities **[NEW]**
- Portals — render a separate world visible through a window in the scene **[NEW]**

### SwiftUI (visionOS)
- `WindowGroup` — standard window scene
- `VolumetricWindowStyle` — fixed-size 3D volume window **[NEW]**
- `ImmersiveSpace` — full or shared space scene **[NEW]**
- Gesture input on entities via SwiftUI gesture system

### CompositorServices **[NEW]**
- `CompositorLayer` — custom Metal rendering surface for fully immersive apps **[NEW]**
- Full access to per-eye rendering with custom Metal shaders and post-processing

### ARKit (visionOS) **[NEW]**
- Hand tracking — raw hand joint data; requires user permission
- Scene understanding — room mesh, plane detection, surface materials; Full Space only; requires permission
- Head position / head tracking for custom spatial audio

### Metal (visionOS)
- Full Metal pipeline available via CompositorServices for fully immersive experiences

### Reality Composer Pro **[NEW]**
- Visual scene assembly tool bundled with Xcode
- Loads and previews USD models
- MaterialX shader graph editor
- Auto-optimizes assets for device at build time

### Networking / Multiplayer
- Web networking, low-level sockets, Multipeer Connectivity — all available
- SharePlay for social/multiplayer experiences
- Cross-platform play: visionOS players with iOS/iPadOS players

## Code Highlights
No code samples are included in this session. It is a strategic overview and framework roadmap.

## Takeaways
- Choose `Volume` (fixed-size box) over `Window` (player-resizable) for 3D game content so you control layout and prevent clipping.
- Adopt `CollisionComponent` + `InputTargetComponent` on any RealityKit entity you want to be interactable via system Look & Tap gestures.
- Dynamic Foveation is free — the platform renderer handles it automatically for any app running in Shared or Full Space.
- For direct-control metal rendering and fully immersive experiences, use `CompositorServices` + ARKit; for everything else, start with RealityKit.

---
_Source: WWDC23 Session 10096 page (abstract, chapter summaries, and resource links)._
