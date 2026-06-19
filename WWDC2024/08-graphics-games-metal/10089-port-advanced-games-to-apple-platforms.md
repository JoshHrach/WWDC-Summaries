# Port Advanced Games to Apple Platforms
**WWDC24 · Session 10089** · [Watch](https://developer.apple.com/videos/play/wwdc2024/10089/)

_Platforms:_ macOS Sequoia 15, iOS 18, iPadOS 18

## Overview
This session presents the full porting pipeline for bringing advanced Windows and console-quality games to Mac, iPhone, and iPad using Game Porting Toolkit 2 (GPTK2). It covers everything from initial evaluation of a Windows binary on Apple silicon through shader conversion, graphics porting with Metal, input and audio integration, cloud saves, and final debugging and profiling with updated Metal tools.

Game Porting Toolkit 2 is a major update that adds AVX2 instruction support, ray tracing compatibility, improved graphics and compute compatibility, and a brand-new interactive sample code project. Every step of the porting journey — evaluate, port, debug — is addressed with dedicated tooling.

The session also introduces several new Metal capabilities specifically designed to ease cross-platform game development, including Metal residency sets, unified Metal shaders deployable across macOS and iOS from a single compile, and enhanced MetalFX upscaling with a reactive mask.

## Key Topics

**Evaluate Your Game**
The updated evaluation environment for Windows games now supports AVX instructions and ray tracing. Developers can run their existing Windows executable on Apple silicon before writing a single line of ported code to gauge performance and shader compatibility.

**Game Porting Example Code**
GPTK2 ships a new interactive tutorial sample project. Organized by subsystem (graphics, shaders, audio, game controllers, cloud saves, etc.), each folder contains a README and all associated code. The sample targets both macOS and iOS out of the box and is pre-configured with Metal-cpp.

**Project Configuration**
Xcode multi-destination setup targeting macOS and iOS simultaneously. Target conditionals (`#if os(iOS)` / `#if os(macOS)`) allow per-OS code paths. SDK-based file filters handle lifecycle differences. Game Mode comes to iOS 18 — opt in via `GCSupportsGameMode` key in Info.plist. Metal device initialization is now unified across macOS and iOS.

**Shader Conversion with Metal Shader Converter**
Metal shader converter (part of GPTK2) converts HLSL shaders — including ray tracing, mesh shaders, geometry, and tessellation — to MetalIR. New this year: globally-coherent texture access support, full debug information propagation from HLSL source, and compile-once/deploy-everywhere unified shaders for both macOS and iOS.

**Metal Residency Sets**
New API replacing per-resource residency management. Developers create a `MTLResidencySet`, add textures, buffers, and heaps in bulk, commit changes, and associate the set with a command queue or individual command buffers. Simplifies ray tracing adoption significantly.

**MetalFX Upscaling**
New reactive mask (`setReactiveMaskTexture`) for better upscaling fidelity on fast-moving alpha-blended objects.

**Input, Rumble, Audio, Cloud Saves, Game Center**
Game Controller framework used for unified input polling across macOS and iOS. Core Haptics + GCController for rumble. PHASE framework for spatial audio. CloudKit (`CloudSaveManager`) for cross-device save sync. `GKAchievement.reportAchievements` for Game Center achievements.

**Metal Tools for Ported Shaders**
API validation (improved for shader converter argument buffer samplers), shader validation (new texture-type mismatch check, per-pipeline enable/disable), shader debugger (full HLSL source-level variable inspection), and shader profiler (cost graph, performance heat map, SIMD group execution history) all now work with HLSL-converted shaders.

## APIs & Frameworks

**Metal**
- `MTLResidencySet` — **[NEW]** bulk residency management
- `MTLResidencySetDescriptor` — **[NEW]**
- `MTLCommandQueue.addResidencySet(_:)` — **[NEW]**
- `MTLResidencySet.addAllocation(_:)` / `commit()` — **[NEW]**
- `MTLRenderPassDescriptor` — color/depth attachment clear configuration
- `MTLCommandBuffer` — inherits residency sets from queue
- Globally-coherent texture access — **[NEW]**
- Row-major acceleration structure transformation matrices — **[NEW]**
- Direct on-chip intersection result storage — **[NEW]**
- Unified Metal shaders (compile once, deploy to macOS + iOS) — **[NEW]**

**MetalFX**
- `MTLFXTemporalScaler` — `setReactiveMaskTexture(_:)` reactive mask **[NEW]**
- `setColorTexture`, `setDepthTexture`, `setMotionTexture`, `setOutputTexture`
- `setJitterOffsetX/Y`, `encodeToCommandBuffer(_:)`

**Metal Shader Converter (GPTK2)**
- `metal-shaderconverter` command-line tool — **[NEW/UPDATED]**
- Runtime header-only library for resource binding
- HLSL debug info propagation (`-Zi -Qembed_debug` DXC flags)

**Metal-cpp**
- C++ bindings for Metal, now bundled in GPTK2 — **[NEW in GPTK2]**

**Game Controller Framework**
- `GCController` — unified input across macOS/iOS
- `GCSupportsGameMode` Info.plist key (iOS 18 Game Mode opt-in) — **[NEW]**

**Core Haptics**
- `CHHapticEngine` — created via `GCController.haptics` property

**PHASE (Physical Audio Spatialization Engine)**
- Spatial audio with geometric occlusion simulation

**CloudKit**
- `CloudSaveManager` (sample class) — `sync(completionHandler:)`, `upload(completionHandler:)`

**Game Center / GameKit**
- `GKAchievement.reportAchievements(_:withCompletionHandler:)`

**Device Certification API**
- Query device performance profiles to scale game configuration — **[NEW]**

**Compositor Services / ARKit** (referenced for visionOS context)

**Metal Debugger / Instruments**
- API validation (per-pipeline control) — **[NEW]**
- Shader validation with texture-type mismatch detection — **[NEW]**
- Shader profiler cost graph and performance heat map — **[NEW for HLSL shaders]**
- Shader debugger with HLSL source-level inspection — **[NEW for HLSL shaders]**

## Code Highlights

Build a residency set (Metal-cpp):
```cpp
MTL::ResidencySet* residencySet = device->newResidencySet(residencySetDescriptor, &error);
commandQueue->addResidencySet(residencySet);
residencySet->addAllocation(texture);
residencySet->addAllocation(buffer);
residencySet->addAllocation(heap);
residencySet->commit();
// At draw time: commit commandBuffer — residency is inherited automatically
```

MetalFX reactive mask upscaling:
```cpp
mfxTemporalScaler->setReactiveMaskTexture(currentFrameReactiveMask);
mfxTemporalScaler->encodeToCommandBuffer(commandBuffer);
```

## Takeaways
- Download GPTK2 from the new landing page at developer.apple.com/games/game-porting-toolkit and start with the interactive sample code project.
- Use Metal residency sets to simplify GPU memory management and enable ray tracing adoption.
- Compile HLSL shaders once with Metal shader converter (using `-Zi -Qembed_debug`) and deploy to both macOS and iOS.
- Enable per-pipeline shader validation to isolate issues in specific pipelines without global performance impact.

---
_Source: WWDC24 Session 10089 page (abstract, chapter summaries, code samples, and resource links)._
