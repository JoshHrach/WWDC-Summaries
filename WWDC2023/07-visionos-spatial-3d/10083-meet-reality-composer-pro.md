# Meet Reality Composer Pro
**WWDC23 · Session 10083** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10083/)

_Platforms:_ visionOS 1, macOS Sonoma 14

## Overview
Reality Composer Pro is a brand-new macOS developer tool introduced alongside visionOS for composing, editing, and previewing 3D spatial content. It replaces and greatly extends the original Reality Composer app, targeting professional development workflows. Projects are structured as Swift packages containing USD (Universal Scene Description) files organized within an `.rkassets` folder; during build, Xcode compiles these assets into a framework that visionOS apps can consume.

The tool integrates tightly with Xcode: the visionOS app template automatically generates a linked Reality Composer Pro package, and developers can open and edit it directly from Xcode. The session walks through composing a Dioramas app scene from terrain meshes, location pins, particle-emitter clouds, spatially-located bird audio, and performance optimization via the Statistics panel.

On-device preview lets developers connect Apple Vision Pro to their Mac and see the composed scene in AR in real time, with pinch-to-rotate and pinch-to-scale gestures.

## Key Topics

### Project Setup and Structure
- Two setup paths: standalone Reality Composer Pro project, or Xcode visionOS app template (recommended) which generates and links a Swift package
- 3D content files are USD / USDA / USDZ; all files in the `.rkassets` folder are compiled into a framework at build time
- Swift package structure enables clean dependency management and build-time asset processing

### UI Navigation
- **Viewport** – visualize and navigate scenes; keyboard (WASD / arrow keys) and optional game controller navigation
- **Hierarchy panel** – search, select, organize 3D objects in the scene tree
- **Inspector panel** – edit position, rotation, scale, and any RealityKit Component properties; Add Component button for built-in components
- **Editor panel tabs**: Project Browser, Shader Graph, Audio Mixer, Statistics

### Composing Scenes
- Three asset sources: import from disk, Content Library (curated Apple assets), Object Capture
- Assets are added as USD references (not copies); updating the source asset propagates everywhere
- Multiple scenes per project: scenes can be full compositions or reusable pieces referenced by other scenes
- Grouping objects in the hierarchy for organizational purposes; Command-D to duplicate

### Particle Emitters
- Particle Emitter Component added via hierarchy panel (plus button) or Add Component in inspector
- Two sections: **Particles** (appearance of individual particles) and **Emitter** (how particles are spawned)
- Built-in presets as starting points (Impact, Fire, Rain, etc.)
- Emitter properties: Timing (idle duration), Shape (cylinder, sphere, etc.), birth location (surface vs. volume), isLocalSpace toggle
- Particle properties: birth rate, life span, color, texture, force fields, rendering
- Playback controls in the inspector for preview; Settings to change viewport background

### Audio Authoring
- Three audio source types:
  - **Spatial Audio** – has position and direction in 3D space (e.g., bird's beak)
  - **Ambient Audio** – direction only, no position (e.g., wind)
  - **Channel Audio** – neither position nor direction (e.g., background music)
- **Audio File** – imported audio asset; configurable properties (looping, etc.)
- **Audio File Group** – collection of audio files; plays a random member on each trigger
- Audio sources and groups added via hierarchy panel plus button or Add Component
- Audio playback in Xcode apps requires additional code (covered in "Work with Reality Composer Pro content in Xcode")

### Statistics Panel
- Dedicated performance optimization editor with categories: General, Physics, Animation, Particle Emitters, Audio, Materials, Geometry, Textures
- Expand button on each category shows detailed per-asset breakdown
- Identifies hot spots (e.g., mesh triangle counts per object) enabling targeted optimization

### On-Device Preview
- Connect Apple Vision Pro to Mac, select it in the toolbar dropdown, toggle the on-device preview button
- Scene appears in AR; pinch-drag to rotate, pinch to scale
- Allows real-world spatial scale validation before writing any Xcode code

## APIs & Frameworks

- **Reality Composer Pro** **[NEW]** – macOS developer tool for composing visionOS 3D content
- **USD (Universal Scene Description)** / USDA / USDZ – file format for all 3D content in Reality Composer Pro
- `.rkassets` folder **[NEW]** – designates content for build-time compilation into a framework
- **RealityKit** – underlying 3D engine; components configured in Reality Composer Pro
- `ParticleEmitterComponent` **[NEW]** – RealityKit component for particle systems; configurable in Reality Composer Pro
  - Emitter shape: `.sphere`, `.cylinder`, `.cone`, `.plane`, `.box`, `.point`, `.torus`
  - Birth location: `.surface`, `.volume`
  - `isLocalSpace` property
  - Particle properties: `birthRate`, `lifeSpan`, color, texture, force fields
- `SpatialAudioComponent` **[NEW]** – RealityKit component for 3D-positioned audio sources
- `AmbientAudioComponent` **[NEW]** – RealityKit component for directional but non-positioned audio
- `ChannelAudioComponent` **[NEW]** – RealityKit component for non-spatial background audio
- Audio File Group – Reality Composer Pro construct for randomized audio playback
- Shader Graph editor – node-based material editing (detailed in "Explore materials in Reality Composer Pro")
- Audio Mixer – audio level and effect editing panel
- Statistics panel – scene performance analysis tool
- **Object Capture** integration – import Object Capture photos directly into Reality Composer Pro for Mac reconstruction
- On-device preview – live AR preview on connected Apple Vision Pro device

## Code Highlights

No source code is shown in this session — Reality Composer Pro is a graphical tool. For runtime code to load and interact with the generated framework assets, see "Work with Reality Composer Pro content in Xcode" (session 10273).

Example of the Swift package import that Reality Composer Pro generates for an Xcode project:

```swift
// In Package.swift (auto-generated by visionOS Xcode template)
.package(path: "MyApp_RCP")
// Assets in .rkassets compiled to a framework importable as:
import MyApp_RCP
```

## Takeaways
- Reality Composer Pro's USD-based Swift package architecture cleanly separates 3D asset authoring from app code, and the build system handles asset compilation automatically.
- Particle emitters and spatial/ambient/channel audio sources are first-class components configurable entirely in the tool without writing code.
- The Statistics panel's per-asset triangle and resource breakdown is the primary workflow for catching performance issues early—trim mesh complexity before writing runtime logic.
- On-device preview via a connected Apple Vision Pro is the fastest feedback loop for validating spatial scale and comfort before Xcode builds.

---
_Source: WWDC23 Session 10083 page (abstract, chapter summaries, code samples, and resource links)._
