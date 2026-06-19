# Optimize High-End Games for Apple GPUs
**WWDC21 · Session 10148** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10148/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
This session documents real-world GPU optimization work done with Larian Studios (Baldur's Gate 3, Divinity: Original Sin 2) and 4A Games (Metro Exodus) on Apple GPUs. It presents a repeatable four-step methodology: measure, set targets, analyze, improve and verify. Working through shader optimization, lossless compression, workload overlap, ring-buffer management, shader compiler flags, and redundant bindings, the teams achieved 33% frame-time improvement for BG3 and 15% for Metro Exodus.

Xcode 13 introduces the new GPU Timeline in Metal Debugger — a performance view showing vertex, fragment, and compute pipeline stages running in parallel, along with shader occupancy, bandwidth, and performance limiter counters. The demo on Divinity: Original Sin 2 shows a 30% ambient-occlusion shader improvement by switching SSAO from F32-only to a mixed F32/F16 approach, reducing register pressure and nearly doubling shader occupancy.

The session emphasizes the importance of profiling tools (Metal Debugger, Metal System Trace, Game Performance template in Instruments) before making changes, and provides concrete Metal API techniques developers can apply directly.

## Key Topics
- **Optimization Methodology** — Measure, set targets, analyze, improve/verify; repeat until targets met; save GPU trace and Instruments trace for before/after comparison.
- **Shader Optimization (BG3)** — Split a complex compute shader with 4,500+ instructions into targeted permutations; use half-precision (`half`) types to reduce register pressure; result: 84% instruction reduction, 8 ms GPU time saved.
- **Lossless Compression (BG3)** — Avoid `MTLTextureUsage.shaderWrite` and `pixelFormatView` flags unless required; use texture views with swizzle patterns for component reordering; texture views for linear-to-sRGB don't need `pixelFormatView`.
- **Workload Overlap (BG3)** — Reorder frame graph tasks to increase vertex/fragment/compute channel overlap; move CascadedShadowBuffer stage earlier for 1 ms win; cross-frame overlap on TBDR GPUs.
- **Ring Buffer / Unified Memory (BG3)** — On devices with unified memory (`MTLDevice.hasUnifiedMemory`), skip private buffer + blit encoder; add an extra ring-buffer slot to avoid completion-handler wait; cross-frame TBDR overlap achieved.
- **Fast Math Compiler Flag (Metro Exodus)** — Ensure Metal shader compiler's fast math flag is enabled; disabling it suppresses algebraic-equivalent optimizations; 21% frame-time decrease after enabling.
- **Redundant Bindings (Metro Exodus)** — Pre-cache resource bindings and call `setFragmentTextures:withRange:` in a single call only when bindings changed; 30–50% encoding time reduction, up to 3 ms GPU time reduction.
- **GPU Timeline (Xcode 13, NEW)** — New Performance page in Metal Debugger; separate tracks per pipeline stage; Shader Timeline shows individual shader execution; load/store action tracks; per-shader compiler stats and runtime metrics; "Reload Shaders" for live shader editing.

## APIs & Frameworks
- **Metal** framework
  - `MTLDevice.hasUnifiedMemory` — Bool; true on Apple Silicon and T-series Macs; skip private buffer blit on unified memory
  - `MTLBuffer` (`MTLStorageModeShared`, `MTLStorageModePrivate`) — Storage mode selection based on memory architecture
  - `MTLCommandBuffer.addCompletedHandler(_:)` — Completion callback; use to guard shared buffer writes
  - `MTLRenderCommandEncoder.setFragmentTextures(_:with:)` — Batch-set multiple fragment textures in one call (range-based)
  - `MTLTextureUsage.shaderWrite` — Disables lossless compression; use only when compute/fragment `write()` is needed
  - `MTLTextureUsage.pixelFormatView` — Disables lossless compression; use texture view with swizzle instead
  - `MTLTexture.makeTextureView(pixelFormat:textureType:levels:slices:swizzle:)` — Create a view with swizzle pattern instead of `pixelFormatView` flag
  - `MTLRenderPassDescriptor` — Load/store actions visible in GPU Timeline load/store tracks
  - Metal Shading Language: `half` type — 16-bit float; half the register usage of `float`; use where full F32 precision not required
  - Metal compiler option: `-ffast-math` / fast math flag — Enables algebraic transformations; on by default; verify not disabled in cross-platform shader translation layers
- **Metal Debugger** (Xcode 13)
  - GPU Timeline **[NEW]** — Performance page with per-pipeline-stage tracks, shader occupancy, bandwidth, limiter counters
  - Shader Timeline **[NEW]** — Per-shader execution tracks within an encoder; load/store action tracks
  - Bandwidth Insights — Lossless compression warnings; flags textures that can't be compressed
  - API Insights — Redundant binding warnings
  - Show Dependencies — Render pass dependency graph visualization
  - "Reload Shaders" button **[NEW]** — Live shader recompile and reprof from source editor
- **Instruments**
  - Metal System Trace template — GPU execution and scheduling analysis, frame-over-frame; vertex/fragment/compute channel tracks
  - Game Performance template — Extends Metal System Trace with thread stall and thermal notification analysis

## Code Highlights
Choose shared vs. private buffer count based on unified memory:

```cpp
static const uint32_t MAX_FRAMES_IN_FLIGHT = 3;
uint32_t sharedBuffersCount  = 0;
uint32_t privateBuffersCount = 0;
if (device.hasUnifiedMemory) {
    sharedBuffersCount = MAX_FRAMES_IN_FLIGHT + 1; // extra buffer avoids wait
    privateBuffersCount = 0;
} else {
    sharedBuffersCount  = MAX_FRAMES_IN_FLIGHT;
    privateBuffersCount = 1;
}
```

Avoid redundant fragment texture bindings:

```cpp
void Renderer::SetFragmentTexture(uint32_t index, id<MTLTexture> texture) {
    if (m_FragmentTextures[index] != texture) {
        m_FragmentTextures[index] = texture;
        m_FragmentTexturesChanged = true;
    }
}
void Renderer::BindFragmentTextures() {
    if (m_FragmentTexturesChanged) {
        [m_RenderCommandEncoder setFragmentTextures:m_FragmentTextures
                                          withRange:NSMakeRange(0, m_LastFragmentTexture + 1)];
        m_FragmentTexturesChanged = false;
    }
}
```

## Takeaways
- Split expensive compute shaders into focused permutations and use `half` types to reduce register pressure and increase parallelism.
- Avoid `MTLTextureUsage.shaderWrite` and `pixelFormatView` unless strictly necessary; they disable lossless compression and increase bandwidth.
- On unified-memory devices, skip the private-buffer blit pattern and add one extra shared ring-buffer slot to prevent data races without stalling.
- Always verify Metal's fast math flag is enabled in your shader build pipeline, especially when using a cross-platform translation layer.

---
_Source: WWDC21 Session 10148 page (abstract, chapter summaries, code samples, and resource links)._
