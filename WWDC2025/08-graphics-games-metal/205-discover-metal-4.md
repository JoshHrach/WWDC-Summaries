# Discover Metal 4
**WWDC25 · Session 205** · [Watch](https://developer.apple.com/videos/play/wwdc2025/205/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26, visionOS 26 (requires Apple M1+ or A14 Bionic+)

## Overview
Metal 4 is a major, backwards-compatible evolution of Apple's low-level GPU API, designed for the next generation of demanding games and pro apps. Built on the same `MTLDevice` as existing Metal code, it introduces an entirely new command encoding model with explicit memory management (`MTL4CommandAllocator`), a bindless resource model (`MTL4ArgumentTable` + residency sets), faster and more controllable shader compilation (`MTL4Compiler` + flexible render pipeline states), native tensor types and a dedicated machine learning command encoder, and MetalFX frame interpolation and denoising.

Adoption is modular: developers can port compilation, command encoding, resource management, and ML integration independently and incrementally.

## Key Topics

### Command Encoding
- **`MTL4CommandQueue`** — obtained from `MTLDevice`; decoupled from command buffers.
- **`MTL4CommandBuffer`** — independent of the queue; easily encoded in parallel.
- **`MTL4CommandAllocator`** — explicit control of command buffer memory allocation.
- **Unified compute encoder** — handles compute dispatches, blits, and acceleration-structure builds in a single encoder (fewer encoder boundaries).
- **`MTL4RenderCommandEncoder`** — new attachment map maps logical shader outputs to physical color attachments; swap color attachments on the fly without allocating new encoders.

### Resource Management
- **`MTL4ArgumentTable`** — stores binding points; sized to only what an app needs; can be shared across shader stages. Enables bindless patterns.
- **Residency sets** — explicitly declare which resources the GPU can access; populate at startup, add to the command queue, update on background threads. Eliminates per-draw residency overhead.
- **Placement sparse resources** — allocate buffers/textures without physical pages; map pages from a `MTLHeap` on demand for fine-grained memory control and streaming.
- **Barrier API** — explicit stage-to-stage synchronization (e.g., dispatch → fragment) replacing implicit hazard tracking.

### Shader Compilation
- **`MTL4Compiler`** — separate from `MTLDevice`; inherits Quality-of-Service class from the requesting thread so high-priority compilations preempt lower-priority ones.
- **Flexible render pipeline states** — compile common Metal IR once (unspecialized pipeline), then specialize for different color states by re-using the compiled IR. Reduces redundant on-device compilation.
- Harvesting improvements for ahead-of-time pipeline compilation workflows.

### Machine Learning Integration
- **Tensors** — multi-dimensional data containers natively in the Metal API and Metal Shading Language (MSL). **[NEW]**
- **Machine learning command encoder** — executes large Core ML networks (converted to Metal package format) within a Metal command buffer; supports argument tables and barriers like other encoders. **[NEW]**
- **Metal performance primitives** — shader-level tensor ops (MSL functions) inlined and optimized by the compiler; ideal for small networks embedded in shaders (e.g., neural material evaluation). **[NEW]**

### MetalFX Updates
- **Frame interpolation** — generates intermediate frames in much less time than full rendering; enables higher refresh rates. **[NEW]**
- **Denoising during upscale** — removes ray-tracing noise during MetalFX upscaling. **[NEW]**
- Existing spatial/temporal upscaling unchanged.

### Adoption Strategy
- Use the same `MTLDevice`; adopt Metal 4 objects incrementally.
- Suggested order: Compiler → Command encoding → Resource management (residency sets) → ML.
- Xcode 26 includes a new Metal 4 project template.
- Existing Metal developer tools (API Validation, Shader Validation, Metal Debugger, Metal System Trace, Performance HUD) all support Metal 4.

## APIs & Frameworks

**Metal 4 (all NEW)**
- `MTL4CommandQueue`, `MTL4CommandBuffer`, `MTL4CommandAllocator`
- `MTL4RenderCommandEncoder` (with attachment map)
- Unified compute encoder (blit + compute + AS build)
- `MTL4ArgumentTable`
- Residency sets (`MTLResidencySet`)
- Placement sparse resources for `MTLBuffer` / `MTLTexture`
- Barrier API (stage-to-stage)
- `MTL4Compiler` (with QoS inheritance)
- Flexible render pipeline states (unspecialized → specialized)
- Metal tensors (API + MSL)
- Machine learning command encoder
- Metal performance primitives (MSL tensor ops)

**MetalFX (NEW)**
- Frame interpolation
- Denoising during upscale

## Code Highlights
No single representative snippet was isolated in the session. Sample code is available in the linked documentation: "Drawing a triangle with Metal 4," "Combining blit and compute operations in a single pass," and "Using the Metal 4 compilation API."

## Takeaways
- Metal 4 is backward-compatible — adopt it incrementally starting with `MTL4Compiler` for immediate QoS improvements with minimal code change.
- Residency sets are among the easiest wins: replace per-draw residency calls with a one-time setup that the system manages efficiently.
- The new ML command encoder and Metal tensor types open a direct path to integrating Core ML networks into rendering pipelines (neural shading, asset compression, animation blending).
- Use MetalFX frame interpolation alongside temporal upscaling to hit higher refresh rates without sacrificing image quality.

---
_Source: WWDC25 Session 205 page (abstract, chapter summaries, and resource links)._
