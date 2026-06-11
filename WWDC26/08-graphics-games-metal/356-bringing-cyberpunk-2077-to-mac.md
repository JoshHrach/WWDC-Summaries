# Bringing Cyberpunk 2077 to Mac
**WWDC26 · Session 356** · [Watch](https://developer.apple.com/videos/play/wwdc2026/356/)

_Platforms:_ macOS

## Overview
This session is a behind-the-scenes case study with CD PROJEKT RED's Associate Game Director Paweł Sasko, describing how Cyberpunk 2077: Ultimate Edition was brought natively to macOS. The port represents a new benchmark for AAA gaming on Apple silicon: a dense open-world RPG with path tracing, ray tracing, complex streaming, and a consistent quality bar across the entire Mac hardware lineup.

The team defined a three-part quality bar from the outset—visual fidelity matching the PC release, stable and consistent performance, and a native macOS feel—then executed against it using Apple's Game Porting Toolkit for evaluation, Metal Shader Converter for shader translation, a native Metal rendering path, and MetalFX Upscaling for scalable performance. The port demonstrates that the evaluation-first workflow recommended by Apple is practical at AAA scale, providing real CPU and GPU pressure data before a single line of platform code was written.

The headline feature developed specifically for this port is the "For this Mac" graphics preset, which automatically detects each Mac's hardware configuration and configures an optimal combination of resolution, MetalFX Dynamic Resolution Scaling, FPS target, V-Sync, and HDR settings—providing a great out-of-box experience without requiring players to navigate complex graphics menus.

## Key Topics

### Evaluation with Game Porting Toolkit (4:50)
Before writing any Metal code, the team ran the Windows build in the Game Porting Toolkit's DirectX translation environment. This provided immediate feasibility signals: CPU/GPU load, memory pressure, frame pacing, and frame times across different hardware tiers. Per-thread CPU breakdowns helped identify where the game's engine spent its time under translation, informing the porting roadmap and resourcing decisions. Metal HUD overlays surfaced GPU metrics without modifying the translated build.

### How the Port Was Executed (3:45)
The port comprised: a native macOS/Apple silicon build replacing the Windows binaries; an adapted data pipeline for asset streaming; a thin architecture bridge mapping engine abstractions to Metal; Metal Shader Converter to translate the existing HLSL shader library to Metal IR without a full rewrite; a native Metal rendering foundation integrating with the engine's render graph; and MetalFX Upscaling for temporal super-resolution across device tiers.

### "For this Mac" Preset (13:16)
A device-based auto-configuration system that reads the Mac's hardware identifier at launch and looks up a curated set of graphics settings—target FPS (30 or 60), MetalFX Dynamic Resolution Scaling bounds (minimum and maximum resolution fractions), V-Sync mode, HDR on/off—tuned specifically for each Mac model in the lineup. This is demoed live in the in-game Dogtown district and shown working across entry-level, mid-range, and high-end Mac configurations. The preset eliminates the "bad first impression" problem where players launch a demanding game with default PC settings and get poor performance.

### Native Platform Feel (13:05)
Beyond rendering, the team invested in macOS-native behaviors: proper window management and app switching behavior, full-screen and windowed modes with correct macOS chrome, native input handling for trackpad and keyboard, macOS-standard HID device support, Spatial Audio via PHASE for headphone playback, and cloud saves via Game Center.

## APIs & Frameworks

### Game Porting Toolkit
- Game Porting Toolkit (GPTK) — evaluation environment for running DirectX games on macOS before porting
- Metal Performance HUD — GPU metric overlay used during GPTK evaluation phase
- Per-thread CPU breakdown — profiling tool used to identify engine hotspots under translation

### Metal
- Metal — native GPU API used as the rendering foundation
- Metal Shader Converter — translates HLSL shaders to Metal IR for native compilation
- Native Metal rendering path — replaced the DirectX rendering backend in the engine

### MetalFX
- `MTLFXTemporalScaler` / MetalFX Upscaling — temporal super-resolution used for scalable performance
- MetalFX Dynamic Resolution Scaling (DRS) — dynamic adjustment of internal render resolution within configured bounds
- MetalFX-related metrics in Metal HUD — FPS, upscaling ratio, and related overlays

### Audio
- PHASE (Physical Audio Spatialization Engine) — Spatial Audio framework used for headphone playback
- Personalized Spatial Audio support via PHASE

### Additional Platform APIs
- Game Center — cloud saves integration
- HID / GCController framework — native input device handling
- Metal tone mapping — custom tone mapper (reference: `Performing your own tone mapping` documentation)

## Code Highlights
No code samples were included in this session (it is a developer story/case study presentation rather than a technical how-to). Key techniques to study via linked resources:

- `Performing your own tone mapping` (Metal documentation) — for implementing HDR output and custom tone mapping as done in the port
- `Personalizing spatial audio in your app` (PHASE documentation) — for the spatial audio integration
- Game Porting Toolkit download — to replicate the evaluation-first workflow

## Takeaways
- The evaluation-first approach (run your Windows build in GPTK before writing code) provides real performance data that de-risks the porting commitment and informs resource planning.
- Metal Shader Converter makes it practical to translate an existing HLSL shader library to Metal without a full rewrite, which was essential for a codebase the scale of Cyberpunk 2077.
- A device-aware "For this Mac" preset that auto-configures MetalFX DRS, resolution bounds, FPS target, and HDR delivers the best first-launch experience and is replicable for any game.
- Native platform feel (windowing, input, audio, cloud saves) distinguishes a port that merely runs from one that feels intentionally designed for macOS.

---
_Source: WWDC26 Session 356 page (abstract, chapter summaries, and resource links)._
