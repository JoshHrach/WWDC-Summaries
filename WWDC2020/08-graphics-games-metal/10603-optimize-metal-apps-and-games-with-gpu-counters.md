# Optimize Metal apps and games with GPU counters
**WWDC20 · Session 10603** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10603/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
GPU performance counters let developers precisely measure GPU utilization to pinpoint bottlenecks and optimize Metal apps and games. This session walks through the tools available in Metal System Trace (Instruments) and the Metal Debugger in Xcode 12, explaining what each category of counter means in the context of Apple's tile-based deferred rendering (TBDR) GPU architecture.

The session covers the Apple GPU pipeline — tiling phase (vertex shading and primitive binning) and rendering phase (per-tile pixel shading) — and explains how each hardware unit (ALU, TPU, Pixel Backend, Tile Memory, GPU LLC) maps to a group of counters. Two live demos using the game Respawnables Heroes illustrate how to record limiters in Instruments and drill into per-draw-call data in Xcode to find concrete optimizations.

A key outcome from the demo: switching texture storage mode to `private` (enabling lossless compression), increasing FP16 usage, and applying ASTC block compression resulted in the game reaching a steady 120 FPS on iPad Pro.

## Key Topics

### Apple GPU Architecture Recap
- Unified memory architecture (CPU + GPU share System Memory); no dedicated video memory
- TBDR: Tiling Phase (vertex shading, primitive binning) then Rendering Phase (per-tile pixel shading)
- GPU core = Shader Core + Texture Unit (TPU) + Pixel Backend + Tile Memory + L1 caches; all cores share GPU LLC

### Performance Limiters
Limiters measure activity and stalls across GPU subsystems. The GPU is only as fast as its slowest part; limiters point to that part.

- **ALU Limiter**: FP16 at 2x rate, FP32 at 1x rate, integer/complex ops at 0.5x or less. Divergent execution wastes cycles. Use `-ffast-math`.
- **Texture Read (TPU) Limiter**: 128-bit formats (RGBA32Float) sample at quarter rate. High anisotropy degrades rate. Use mipmaps, ASTC/PVRTC block compression.
- **Texture Write (Pixel Backend) Limiter**: Written on `StoreActionStore`. Watch pixel size and MSAA sample count; prefer coherent writes.
- **Tile Memory Load/Store Limiter**: Covers Imageblock (programmable blending, tile shaders) and Threadgroup memory (compute). Reduce atomics; align to 16 bytes.
- **Buffer Read/Write Limiter**: `device` for per-vertex/fragment data; `constant` for shared data. Pack tightly, vectorize, avoid device atomics and register spills.
- **GPU Last Level Cache Limiter**: Shared across all cores; caches textures and buffers; handles device atomics. Improve spatial/temporal locality; prefer threadgroup atomics over device atomics.
- **Fragment Input Interpolation Limiter**: Fixed-function; only remedy is reducing vertex attributes passed to fragment shader.

### Memory Bandwidth Counter
Measures System Memory ↔ GPU transfers. Reduce by: efficient load/store actions (only load/store what's needed), texture compression, private storage mode.

### Occupancy Counters
Occupancy = threads executing / total thread capacity. Sum of Compute + Vertex + Fragment Occupancy. Low occupancy is acceptable if resources are fully utilized. Query `maxTotalThreadsPerThreadgroup`, `threadExecutionWidth`, and static threadgroup memory requirements.

### Hidden Surface Removal (HSR) Efficiency
HSR eliminates overdraw for opaque meshes in submission-order-independent fashion. Measure with: pixels rasterized, fragment shader invocations, pixels stored, Pre-Z test fails. Draw order: opaque → alpha test/discard/depth feedback → translucent. Never interleave opaque and non-opaque meshes.

### Tooling
- **Metal System Trace** (Instruments, Game Performance template): CPU + GPU timeline, per-encoder GPU counters, new Shader Timeline showing which shaders run at each sample point
- **Metal Debugger** (Xcode 12): Per-encoder and per-draw-call counters, counter groups (Memory, Limiters, etc.), custom counter groups, sortable detail table

## APIs & Frameworks

- **Metal** framework
- `MTLCommandBuffer` / `MTLRenderCommandEncoder` — render passes profiled by both tools
- `MTLLoadAction.load` / `MTLStoreAction.store` — trigger TPU reads and Pixel Backend writes
- **Tile Shaders** / **Programmable Blending** — access Tile Memory (Imageblock)
- `MTLComputeCommandEncoder` — threadgroup memory (Tile Memory)
- `MTLTexture` storage modes: `.private` **[NEW advantage]**, `.shared`
- `MTLPixelFormat` — RGBA32Float (quarter rate), RGBA16Float, ASTC/PVRTC block-compressed formats **[NEW: ASTC HDR on A13+]**
- MSAA (`MTLRenderPassDescriptor` sample count)
- `MTLBuffer` address spaces: `device` (read-write), `constant` (read-only)
- Metal Shader Language flag: `-ffast-math`
- SIMD lane operations / threadgroup parallel reductions (alternatives to device atomics)
- `MTLComputePipelineState` properties: `maxTotalThreadsPerThreadgroup`, `threadExecutionWidth`
- **Metal System Trace** instrument (Instruments.app) — GPU counter sets: Performance Limiters, Shader Timeline **[NEW]**
- **Metal Debugger** (Xcode 12) — per-draw-call counters **[NEW]**, counter groups **[NEW]**, custom counter groups **[NEW]**
- `MTLBlitCommandEncoder` — explicit GPU optimization of textures with shared storage mode

## Code Highlights

No standalone code samples were provided; key patterns discussed:

- Set texture storage mode to `.private` to enable lossless GPU texture compression automatically
- Use `constant` address space in Metal shaders for uniform/shared data; `device` for per-vertex/fragment indexed data
- Align threadgroup memory allocations to 16 bytes
- Compile shaders with `-ffast-math` and prefer `half` (FP16) types over `float` (FP32)

## Takeaways

- Always check Performance Limiters first — they reveal the slowest GPU subsystem and guide which counters to investigate next.
- Texture compression (ASTC block compression for assets, lossless via `.private` storage for runtime textures) is the single highest-impact bandwidth optimization on Apple GPUs.
- The new Shader Timeline in Instruments and per-draw-call counters in Xcode 12 make it straightforward to isolate the exact draw call and resource causing a bottleneck.
- Draw geometry in the correct order (opaque before translucent) and avoid divergent execution in shaders to maximize HSR efficiency and ALU throughput.

---
_Source: WWDC20 Session 10603 page (abstract, chapter summaries, code samples, and resource links)._
