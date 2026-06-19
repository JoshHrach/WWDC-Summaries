# Bring Your Game to Mac, Part 1: Make a Game Plan
**WWDC23 · Session 10123** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10123/)

_Platforms:_ macOS Sonoma 14, iPadOS 17

## Overview
Part 1 of a three-part series on porting high-end Windows games to Mac using Apple Silicon. This session introduces the **Game Porting Toolkit** — a new translation environment that runs an unmodified Windows DirectX game on macOS, allowing developers to evaluate graphics feature compatibility and baseline performance before beginning a native port. The toolkit translates Intel instructions, Windows API calls (input, audio, networking, file system), and all modern Direct3D graphics features (GPU-driven pipelines, SIMD, tessellation, geometry shaders) into Metal.

The session also covers the high-level porting roadmap across all five areas: shaders (covered in Part 2), Metal rendering (Part 3), input (Game Controller framework), audio (PHASE, AVAudioEngine, Wwise/FMOD/Unity middleware), and display/HDR (CAMetalLayer, CAMetalDisplayLink, EDR).

## Key Topics

### Game Porting Toolkit
- **New in 2023**: runs an existing unmodified Windows DirectX game on macOS via an emulation/translation layer.
- Translates: Intel x86 instructions, Direct3D 11/12 APIs, keyboard/mouse/controller input, audio, networking, file system.
- Supports all modern graphics features: GPU-driven pipelines, tessellation, geometry shaders, mesh shaders, ray tracing, SIMD.
- Allows developers to see their game running on Mac early in the porting process — before recompiling source, converting shaders, or reimplementing graphics.
- Available at developer.apple.com as a downloadable package.

### Evaluate Your Game with the Metal Performance HUD
- Launch the Windows build in the Game Porting Toolkit from a terminal; logging appears with potential issues.
- The **Metal Performance HUD** is enhanced in the toolkit to show: GPU/resolution info, frame rate, present-to-present interval, GPU time, memory usage, plus API translation details (Direct3D version, render encoder count, command buffer count, geometry shader count, tessellation shader count, resource copies/clears).
- Use **Instruments Metal System Trace** to investigate frame drops found via the HUD.
- Performance in the translated environment includes translation overhead — native Metal performance will be significantly better.

### Metal Shader Converter
- Automatically converts HLSL GPU shaders to Metal — including geometry, tessellation, mesh, and ray tracing stages.
- Available as a standalone download (Mac and Windows).
- Deep-dive in Part 2: "Compile Your Shaders."

### Rendering with Metal
- Metal 3 features: MetalFX upscaling, fast resource loading, offline compilation, mesh shaders, ray tracing.
- Deep-dive in Part 3: "Render with Metal."

### Input
- **Game Controller framework** — thread-safe, low-latency input across game pads, keyboards, mice, racing wheels, arcade sticks.
- Supports haptics, motion sensors, per-app remapping (system-level), screenshots, video captures, 15-second highlights.
- Direct port from XInput/GameInput Windows APIs; cross-platform engines with existing plug-ins require minimal changes.

### Audio
- Apple devices have state-of-the-art speakers with no fan noise; excellent peripheral support with low-latency spatial audio.
- Cross-platform middleware (Wwise, FMOD, Unity audio) provides native Apple Silicon SDKs — little or no work required.
- For direct API use: `PHASE` (Spatial Mixer Audio Unit), `AVAudioEngine`, `AUSpatialMixer`.
- PHASE supports spatial audio from multi-channel PCM (e.g., 12-channel 7.1.4) data.

### Display and HDR
- `CAMetalLayer` — extended dynamic range (EDR) support for HDR and tone mapping.
- `CAMetalDisplayLink` **[NEW]** — fine-grained control for lowest input and display latency.
- Replaces Windows `AdvancedColorInfo` / `IDXGISwapChain` timing APIs.
- Supports floating-point range, 10-bit integer range, HDR10, PQ10 shaders.

## APIs & Frameworks

### Game Porting Toolkit **[NEW]**
- Game Porting Toolkit — downloadable translation environment; runs unmodified Windows DirectX game on macOS
- Metal Shader Converter — HLSL-to-Metal shader conversion tool (Mac and Windows download) **[NEW]**

### Metal / Metal 3
- Metal — Apple's GPU framework for Mac, iOS, iPadOS
- MetalFX — upscaling (temporal and spatial anti-aliasing); boost frame rate
- MetalFX upscaling **[Metal 3]**
- Fast resource loading **[Metal 3]**
- Offline compilation **[Metal 3]**
- Mesh shaders **[Metal 3]**
- Ray tracing
- `CAMetalLayer` — Metal-backed display layer
- `CAMetalDisplayLink` — frame pacing and display timing API **[NEW]**
- Metal Performance HUD — enhanced in Game Porting Toolkit with API translation metrics

### Instruments
- Metal System Trace — performance capture tool for investigating frame drops

### Game Controller Framework
- `GameController` framework — cross-platform input framework
- Supports: game pads, keyboards, mice, racing wheels, arcade sticks
- Haptics, motion sensors, per-app input remapping
- System screenshot, video capture, 15-second gameplay highlights

### Audio
- `PHASE` (Physical Audio Spatialization Engine) — spatial audio framework
- `AVAudioEngine` — audio engine framework
- `AUSpatialMixer` — Audio Unit for spatial mixing (7.1.4 / 12-channel PCM)
- `CoreAudio` — low-level audio
- `AudioUnit` — audio unit framework
- Wwise, FMOD, Unity audio — third-party middleware with native Apple Silicon SDKs

### Display / HDR
- `CAMetalLayer.wantsExtendedDynamicRangeContent` — enable EDR
- `CAMetalLayer` pixel formats for HDR: `rgba16Float`, `bgr10a2Unorm`, `bgra10_xr`, `bgra10_xr_srgb`
- `CAMetalDisplayLink` **[NEW]** — smooth frame pacing, lowest latency display presentation

## Code Highlights
No code samples are included in this session. The session is a strategic/planning overview directing developers to the toolkit downloads and related sessions.

## Takeaways
- The Game Porting Toolkit lets you run an unmodified Windows DirectX game on macOS to evaluate graphics feature compatibility and baseline performance before starting a native port.
- Metal Shader Converter dramatically reduces the time to port HLSL shaders — previously one of the most time-consuming parts of a game port.
- Audio middleware (Wwise, FMOD, Unity) already supports native Apple Silicon — typically no audio porting work required.
- `CAMetalDisplayLink` (new) provides fine-grained frame pacing control replacing DXGI swap chain timing APIs.

---
_Source: WWDC23 Session 10123 page (abstract, chapter summaries, code samples, and resource links)._
