# Design Advanced Games for Apple Platforms
**WWDC24 · Session 10085** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10085/)

_Platforms:_ iOS 18, iPadOS 18, macOS 15, tvOS 18, visionOS 2

## Overview
This session is a high-level design and architecture guide for developers bringing console- and PC-quality games to Apple platforms. Rather than focusing on a single new API, it surveys the breadth of Apple's gaming stack — Metal, Game Controller, Game Center, Apple Game Porting Toolkit 2, and MetalFX — and gives practical guidance on how to structure a port or a ground-up Apple-native game for maximum performance and platform fit.

The talk is organized around four pillars: input handling, graphics quality, networking and social features, and platform-specific design considerations (especially for Apple Vision Pro). Each pillar includes common mistakes and the recommended pattern to avoid them.

## Key Topics
- **Apple Game Porting Toolkit 2** — an updated translation layer for porting Windows/DirectX games; gains improved coverage of DirectX 12 features and better shader translation fidelity.
- **MetalFX upscaling** — `MTLFXSpatialScaler` (spatial) and `MTLFXTemporalScaler` (temporal with motion vectors) for resolution upscaling; recommend running the game render at 50–66% of native and upscaling to full res.
- **Game Controller framework** — `GCController`, `GCExtendedGamepad`, `GCVirtualController` for software gamepads; trigger haptics via `GCDeviceHaptics` and `CHHapticEngine`.
- **Game Center** — `GKLeaderboard`, `GKAchievement`, `GKMatch` / `GKTurnBasedMatch` for matchmaking; `GKAccessPoint` for the standard Game Center UI entry point.
- **visionOS game design** — fully immersive spaces with RealityKit + Metal; spatial audio through `PHASE`; hand tracking and spatial tap gestures as alternative input; comfort guidelines for movement.
- **Instruments profiling** — Metal System Trace, GPU Timeline, and the new Metal Debugger counters for identifying GPU bottlenecks.

## APIs & Frameworks

**Metal**
- `MTLDevice` — primary GPU interface; unchanged
- `MTLCommandBuffer`, `MTLRenderCommandEncoder`, `MTLComputeCommandEncoder` — unchanged
- `MTLRasterizationRateMap` — variable-rate shading for foveated rendering on visionOS
- `MTLCounterSampleBuffer` — GPU performance counters; new Metal Debugger integration in Xcode 16

**MetalFX**
- `MTLFXSpatialScalerDescriptor` / `MTLFXSpatialScaler` — spatial upscaling (single-frame)
- `MTLFXTemporalScalerDescriptor` / `MTLFXTemporalScaler` — temporal upscaling (multi-frame, requires motion vectors and depth)
- **[NEW]** `MTLFXTemporalScaler` — improved motion vector handling in macOS 15 / iOS 18 for reduced ghosting

**Game Controller**
- `GCController` — physical controller; `GCExtendedGamepad` for full thumbstick + trigger layout
- `GCVirtualController` — on-screen software gamepad for touch devices
- `GCExtendedGamepad.buttonMenu`, `.leftThumbstick`, `.rightTrigger` etc. — input element accessors
- `GCDeviceHaptics` / `CHHapticEngine` — trigger rumble and advanced haptic patterns on supported controllers
- `GCController.current` — observe connected controller changes

**Game Center**
- `GKLocalPlayer` — authenticate the local player; `GKLocalPlayer.local.authenticateHandler`
- `GKLeaderboard` — leaderboard submit (`submitScore(_:context:player:completionHandler:)`) and load
- `GKAchievement` — report and load achievement progress
- `GKMatchmaker` / `GKMatchRequest` — online matchmaking
- `GKTurnBasedMatch` — asynchronous turn-based multiplayer
- `GKAccessPoint` — standard UI entry for Game Center dashboard
- `GKGameCenterViewController` — present Game Center UI inline

**PHASE (spatial audio)**
- `PHASEEngine` — spatial audio engine; use for first-person and third-person game audio on visionOS

**Apple Game Porting Toolkit 2**
- Command-line tool; not an SDK API; used during development porting workflow

## Code Highlights
Enable MetalFX temporal upscaling:

```swift
let desc = MTLFXTemporalScalerDescriptor()
desc.inputWidth = renderWidth
desc.inputHeight = renderHeight
desc.outputWidth = displayWidth
desc.outputHeight = displayHeight
desc.colorTextureFormat = .bgra8Unorm
desc.depthTextureFormat = .depth32Float
desc.motionTextureFormat = .rg16Float
let scaler = desc.makeTemporalScaler(device: device)!
```

Authenticate with Game Center and submit a score:

```swift
GKLocalPlayer.local.authenticateHandler = { vc, error in … }
GKLeaderboard.submitScore(score, context: 0, player: GKLocalPlayer.local,
                           leaderboardIDs: ["com.myapp.highscore"]) { error in … }
```

## Takeaways
- Use MetalFX temporal upscaling at 50% render resolution as the baseline graphics quality strategy; it delivers near-native image quality at a fraction of the rendering cost on Apple Silicon.
- On visionOS, choose between a windowed `SwiftUI`/`RealityKit` scene and a fully immersive Metal-rendered space early — the architecture differs significantly and migration between the two is difficult.
- Integrate Game Center authentication on first launch; even casual games benefit from `GKLeaderboard` for engagement.
- Profile with Metal System Trace in Instruments before assuming a bottleneck — GPU vertex, fragment, and tile shader stages have distinct counters; fixing the wrong stage wastes time.

---
_Source: WWDC24 Session 10085 page (abstract, chapter summaries, code samples, and resource links)._
