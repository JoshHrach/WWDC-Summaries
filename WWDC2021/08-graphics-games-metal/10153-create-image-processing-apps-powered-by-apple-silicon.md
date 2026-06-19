# Create Image Processing Apps Powered by Apple Silicon
**WWDC21 · Session 10153** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10153/)

_Platforms:_ macOS Monterey 12 (Apple silicon / M1)

## Overview
Image-processing apps historically designed around discrete GPUs (with separate system memory and VRAM) need to be rearchitected to fully benefit from Apple silicon's Unified Memory Architecture (UMA) and Tile Based Deferred Renderer (TBDR) GPU. This session presents six concrete optimizations — organized as a progression from eliminating unnecessary memory copies to restructuring an entire video-processing pipeline — that together achieve a 62% reduction in device memory bandwidth for a representative 4K image-processing graph.

The session covers: eliminating blit copies on UMA; replacing compute dispatches with `MTLRenderCommandEncoder` for per-pixel operations; using memoryless attachments and proper load/store actions; using `function_constants` to reduce uber-shader register pressure; using 16-bit types (half/short); and leveraging `CVPixelBuffer` + `IOSurface` for zero-copy GPU-to-media-engine handoff.

## Key Topics

### 1. Eliminate Unnecessary Blit Copies
On discrete GPUs, CPU-decoded frames must be copied to GPU VRAM before processing and back to system memory afterward. On Apple UMA (M1), CPU and GPU share the same memory — blits for residency are no longer needed. Check for UMA at runtime with `MTLDevice.hasUnifiedMemory` and skip the copy. Leaving unnecessary blits in place wastes memory bandwidth, scheduling time, and requires a separate VRAM allocation.

### 2. Replace Compute with Render Command Encoder
Regular compute dispatches require full memory coherency between every dispatch — each shader's output is written to device memory and re-read by the next. Apple's TBDR tile memory persists across draw calls within a single `MTLRenderCommandEncoder`. Key rule: per-pixel operations with no inter-pixel dependency → fragment shader; threadgroup-scoped operations with neighbor pixel access → tile shading; scatter/gather and convolutions with random access patterns → remain as compute kernels.

Additionally, render encoders unlock lossless bandwidth compression for textures and render targets, which is not available for raw buffer compute.

### 3. Load/Store Actions and Memoryless Attachments
At the start of each render pass, tile memory must be initialized via load actions:
- `.dontCare` — when the entire attachment is overwritten (fastest; do not use this when you need to preserve existing contents)
- `.clear` — efficiently clear to a specified value

At the end, store actions determine which attachments are written back to device memory:
- `.store` — write back (only for attachments needed downstream)
- `.dontCare` — discard (use for transient attachments)

Transient intermediate textures should use `storageMode = .memoryless` — they exist only in tile memory for the duration of the render pass, with zero device memory allocation. This saves hundreds of megabytes per frame for 4K/6K/8K content.

### 4. Function Constants for Uber-Shaders
Uber-shaders with runtime control flow (e.g., `if (inputIsHDR)`) force the GPU to keep registers live for all paths simultaneously, limiting occupancy. Replace runtime flags with `function_constant` specialization constants: the compiler can eliminate dead code paths at pipeline creation time, reducing register count and improving simdgroup concurrency.

### 5. 16-Bit Types (half / short)
Apple GPUs have native 16-bit compute support. Using `half` instead of `float` and `ushort` instead of `uint` where precision allows: uses half the registers (increases occupancy), requires less energy, achieves higher peak rates, and reduces texture storage footprint. Type conversions from float to half are usually free. For 3D LUTs: use FP16 or unsigned short (with LUT range passed to shader) instead of RGBA32F to achieve peak bilinear filtering rates.

### 6. Zero-Copy GPU-to-Media-Engine (UMA)
For video encoding: create a `CVPixelBufferPool` backed by `IOSurface`, obtain a `CVPixelBuffer`, create `MTLTexture` objects from both planes via `CVMetalTextureCache`, render into those Metal textures (which update the underlying IOSurface in place), then hand the `CVPixelBuffer` directly to `AVAssetWriter` / `VTCompressionSession` for hardware HEVC encode. No CPU copy is required between GPU and the media engine.

## APIs & Frameworks

**Metal** (`import Metal`) — macOS 12 / Apple silicon

- `MTLDevice.hasUnifiedMemory: Bool` — check for UMA to skip blit residency copies
- `MTLRenderCommandEncoder` — render pass encoder for fragment-shader-based image processing
- `MTLRenderPassDescriptor` — configure attachments with load/store actions
  - `colorAttachments[n].loadAction: MTLLoadAction` — `.dontCare`, `.clear`, `.load`
  - `colorAttachments[n].storeAction: MTLStoreAction` — `.store`, `.dontCare`, `.multisampleResolve`
- `MTLTextureDescriptor.storageMode = .memoryless` **[Apple GPU]** — tile-only allocation; no device memory
- `MTLTileRenderPipelineDescriptor` / tile shading — for threadgroup-scoped filters (e.g., convolutions on tile memory)
- `function_constant(index)` **[MSL]** — shader specialization; eliminates dead paths at PSO creation
- `MTLFunctionConstantValues` — host-side API to provide function constant values before PSO creation
- `half`, `ushort`, `short` **[MSL 16-bit types]** — reduce register pressure; native on Apple GPUs
- Xcode 13 GPU Frame Debugger: PSO inspector shows per-shader register usage; Summary pane shows lossless compression opt-out warnings

**Core Video / Metal interop** (zero-copy GPU→media engine)
- `CVPixelBufferPool` with `kCVPixelBufferIOSurfacePropertiesKey` — create IOSurface-backed pixel buffer pool
- `CVMetalTextureCache` — create `CVMetalTexture` from `CVPixelBuffer` plane
- `CVMetalTextureGetTexture(_:)` — get `MTLTexture` backed by the IOSurface
- `CVPixelBuffer` — pass directly to `AVAssetWriter` / `VTCompressionSession` for zero-copy HW encode

## Code Highlights

Memoryless intermediate attachment:
```swift
let outputTexture = device.makeTexture(descriptor: outputDescriptor)!

var tempDescriptor = outputDescriptor
tempDescriptor.storageMode = .memoryless
let tempTexture = device.makeTexture(descriptor: tempDescriptor)!

let renderPassDesc = MTLRenderPassDescriptor()
renderPassDesc.colorAttachments[0].texture     = outputTexture
renderPassDesc.colorAttachments[0].loadAction  = .dontCare  // fully overwritten
renderPassDesc.colorAttachments[0].storeAction = .store
renderPassDesc.colorAttachments[1].texture     = tempTexture
renderPassDesc.colorAttachments[1].loadAction  = .clear
renderPassDesc.colorAttachments[1].storeAction = .dontCare  // transient, no store
```

Metal Shading Language: function constants vs. uber-shader:
```metal
// Bad: runtime control flow keeps all register paths live
fragment float4 processPixel(constant ParamsStr* cs [[buffer(0)]]) {
    if (cs->inputIsHDR) { /* HDR path */ }
    if (cs->tonemapEnabled) { /* tonemap path */ }
}

// Good: function_constants allow compiler to eliminate dead paths
constant bool featureAEnabled [[function_constant(0)]];
constant bool featureBEnabled [[function_constant(1)]];
fragment float4 processPixel(...) {
    if (featureAEnabled) { /* A path */ } else { /* not-A path */ }
    if (featureBEnabled) { /* B path */ }
}
```

Tile memory access in fragment pipeline (chained filters):
```metal
typedef struct {
    float4 OPTexture       [[color(0)]];
    float4 IntermediateTex [[color(1)]];
} FragmentIO;

fragment FragmentIO Unpack(RasterizerData in [[stage_in]],
    texture2d<float> srcImage [[texture(0)]]) {
    FragmentIO out;
    out.OPTexture       = /* compute output value */;
    out.IntermediateTex = /* compute intermediate value */;
    return out;
}

fragment FragmentIO CSC(RasterizerData in [[stage_in]], FragmentIO Input) {
    FragmentIO out;
    // Reads from tile memory directly via `Input`
    out.IntermediateTex = /* apply color space conversion to Input */;
    return out;
}
```

## Takeaways
- On Apple UMA, blitting for GPU residency is unnecessary — check `MTLDevice.hasUnifiedMemory` and eliminate all copy-for-residency blits as the first optimization step.
- Per-pixel image filters should move from compute dispatches to fragment shaders within one `MTLRenderCommandEncoder`; chained fragment shaders communicate through tile memory without device memory round-trips.
- Memoryless attachments (`.storageMode = .memoryless`) with `.dontCare` store actions eliminate device memory allocation and write-back for transient intermediate textures, saving hundreds of MB per frame.
- Metal `function_constant` specialization replaces uber-shader runtime branches, reducing register pressure and increasing GPU occupancy.
- Zero-copy GPU-to-HEVC-encode is achievable using `CVMetalTextureCache` + `IOSurface`-backed `CVPixelBuffer` pools, letting Metal render directly into the buffer the media engine encodes.

---
_Source: WWDC21 Session 10153 page (abstract, chapter summaries, code samples, and resource links)._
