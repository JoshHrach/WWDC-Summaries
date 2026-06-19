# Harness Apple GPUs with Metal
**WWDC20 · Session 10602** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10602/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This hardware-focused session (no code) explains the Apple GPU architecture and how Metal is co-designed to exploit it. Part one covers the foundational Tile Based Deferred Renderer (TBDR) pipeline shared by all Apple GPUs: the two-phase tiling/rendering architecture, the Tiled Vertex Buffer, load/store actions, Hidden Surface Removal (HSR), Programmable Blending, Memoryless Render Targets, and the efficient MSAA implementation. Part two covers the modern Apple GPU redesign introduced with A11, adding the Imageblock (a 2D on-chip memory structure) and Tile Shading (programmable mid-pass compute dispatches), and discusses GPU-driven rendering via Argument Buffers and Indirect Command Buffers.

Understanding this architecture is the prerequisite for interpreting Metal Debugger Insights (covered in session 10605) and for all Apple GPU optimization work.

## Key Topics

**Unified Memory Architecture**
- CPU and GPU share the same system memory — no dedicated video memory (VRAM)
- GPU has a dedicated on-chip pool called Tile Memory used for the framebuffer and intermediate data
- Memory bandwidth is the key constraint; TBDR minimizes main memory accesses

**TBDR Two-Phase Pipeline**
- Phase 1 — Tiling: vertex shading for the entire render pass; transformed primitives are binned by tile into the Tiled Vertex Buffer
- Phase 2 — Rendering: each tile is processed independently — load, HSR + rasterization, fragment shading, store
- Tiled Vertex Buffer is mostly opaque; if it overflows, a Partial Render occurs (pass is split mid-frame)
- Tiling phase of render pass N overlaps with rendering phase of render pass N-1 for free pipeline throughput

**Load and Store Actions**
- Executed per tile at the start and end of each render pass
- Load: only load attachments your pass actually reads; use `.clear` or `.dontCare` otherwise
- Store: only store attachments downstream passes read; use `.dontCare` for transient attachments (depth, MSAA multisample buffer)
- Incorrect store actions are the most common cause of unnecessary bandwidth — addressed directly by Metal Debugger Insights

**Hidden Surface Removal (HSR)**
- On-chip depth buffer enables pixel-exact, submission-order-independent visibility tracking
- GPU tracks the frontmost primitive ID per pixel for the entire tile before running any fragment shader
- Translucent (alpha-blended) geometry forces a flush of covered pixels — fragment shading runs immediately
- Optimal draw order: opaque → alpha-tested/discard → translucent; never interleave these categories
- HSR eliminates overdraw for opaque geometry entirely — depth pre-passes are unnecessary and harmful

**Programmable Blending**
- Fragment shaders can read the current pixel value directly from Tile Memory
- Enables multi-pass algorithms (deferred lighting, custom compositing) to be merged into a single pass
- Eliminates the need to store intermediate G-Buffer attachments to main memory and reload them

**Memoryless Render Targets**
- Textures declared with memoryless storage mode are never backed by main memory
- They live entirely in Tile Memory for the duration of the pass
- Use for depth/stencil, MSAA multisample buffers, and intermediate G-Buffer attachments that are never read outside their render pass
- Eliminates both memory bandwidth and memory footprint for those resources

**Efficient MSAA**
- MSAA samples stored in Tile Memory; resolved to the resolve texture when the tile is flushed
- `.store` of the multisample texture is never needed — multisample texture should be `.dontCare` store with memoryless storage
- GPU tracks primitive edges; opaque pixels without edges are blended per-pixel, not per-sample

**Modern Apple GPUs (A11+): Imageblock**
- Imageblock: a 2D data structure in Tile Memory with width, height, and pixel depth
- Accessible from both fragment functions and compute kernels (tile shaders)
- Load/store image data in a single operation — more efficient than per-pixel threadgroup memory access
- Used for explicit G-Buffer management and MSAA sample coverage control

**Modern Apple GPUs (A11+): Tile Shading**
- Tile shaders are compute kernel dispatches that can access the Imageblock mid-render pass
- Dispatches interleave with draw calls in API submission order
- Implicit barrier ensures correctness against surrounding draw calls
- Key use case: Tiled Deferred Rendering — cull lights per tile with a tile shader between G-Buffer fill and lighting accumulation, merging three passes into one

**Imageblock Sample Coverage Control**
- Expose per-sample coverage tracking data to the application
- Enable MSAA resolve at a controlled point mid-pass (e.g., after opaque geometry, before heavy particle blending)
- Resolve is fully programmable — implement custom resolves per attachment type (HDR, linear depth, etc.)

**GPU-Driven Rendering**
- Argument Buffers: scene data (meshes, materials, models) made available on the GPU as indirectly-accessed descriptor arrays; no CPU involvement in traversal
- Indirect Command Buffers: GPU encodes its own draw calls based on culling/LOD decisions made entirely on-device
- Eliminates GPU→CPU→GPU synchronization for occlusion culling and LOD selection
- Reference: "Modern Rendering with Metal" sample code (full GPU-driven render loop with Tile Shaders, Imageblocks, Sparse Textures, Variable Rasterization Rate)

## APIs & Frameworks

### Metal API — Load/Store Actions
- `MTLLoadAction.clear` — initialize attachment to a clear color/value (no memory read)
- `MTLLoadAction.dontCare` — contents undefined; fastest initialization
- `MTLLoadAction.load` — load attachment from main memory
- `MTLStoreAction.dontCare` — discard tile contents; no write to main memory **[use for depth, MSAA sample buffer]**
- `MTLStoreAction.store` — write tile contents to main memory
- `MTLStoreAction.multisampleResolve` — resolve MSAA samples to resolve texture; no need to store samples

### Metal API — Tile Shading and Imageblocks (A11+)
- `[[tile_data]]` MSL attribute — marks an Imageblock field accessible by tile shaders
- `MTLRenderPassDescriptor.tileWidth` / `.tileHeight` — tile dispatch size for tile shaders
- `MTLRenderCommandEncoder.dispatchThreadsPerTile(_:)` — dispatch a tile shader mid-pass
- `MTLRenderPipelineDescriptor.tileFunction` — specify the tile shader kernel

### Metal API — Argument Buffers and GPU-Driven Rendering
- `MTLArgumentEncoder` — encodes resource references into a buffer for indirect GPU access
- `MTLIndirectCommandBuffer` — buffer that the GPU populates with draw/dispatch commands
- `MTLRenderCommandEncoder.executeCommandsInBuffer(_:with:)` — execute an indirect command buffer

### Metal Shading Language
- `imageblock<T>` — Imageblock type in tile functions
- `[[color(n)]]` on Imageblock fields — map fields to color attachments
- `threadgroup T data [[imageblock_data]]` — explicit Imageblock declaration

## Code Highlights
No code samples in this session (hardware architecture talk). Key actionable patterns:

Correct load/store actions for depth in a render pass:
```swift
let renderPassDescriptor = MTLRenderPassDescriptor()
renderPassDescriptor.colorAttachments[0].loadAction = .clear
renderPassDescriptor.colorAttachments[0].storeAction = .store      // keep color

renderPassDescriptor.depthAttachment.loadAction = .clear
renderPassDescriptor.depthAttachment.storeAction = .dontCare       // depth not read downstream

renderPassDescriptor.depthAttachment.texture = depthTexture        // declare as memoryless:
// when creating: descriptor.storageMode = .memoryless
```

MSAA setup with memoryless multisample texture:
```swift
let msaaDesc = MTLTextureDescriptor.texture2DDescriptor(pixelFormat: .bgra8Unorm,
                                                         width: width, height: height,
                                                         mipmapped: false)
msaaDesc.textureType = .type2DMultisample
msaaDesc.sampleCount = 4
msaaDesc.storageMode = .memoryless   // no main-memory backing needed
msaaDesc.usage = [.renderTarget]
let msaaTexture = device.makeTexture(descriptor: msaaDesc)!

let rp = MTLRenderPassDescriptor()
rp.colorAttachments[0].texture = msaaTexture
rp.colorAttachments[0].resolveTexture = resolveTexture
rp.colorAttachments[0].loadAction = .clear
rp.colorAttachments[0].storeAction = .multisampleResolve  // resolve; never store samples
```

## Takeaways
- Every Apple GPU is TBDR — optimizing for it means: correct load/store actions, minimal overdraw (opaque-first draw order), Programmable Blending for multi-pass algorithms, and memoryless textures for transient attachments.
- HSR makes depth pre-passes counterproductive — only opaque geometry is needed in submission order, and HSR handles the rest without explicit sorting.
- The single highest-impact optimization for most Apple GPU apps is changing unnecessary `.store` actions to `.dontCare` for depth, stencil, and MSAA multisample textures — Metal Debugger Insights (session 10605) identifies these automatically.
- Modern Apple GPUs (A11+) add Tile Shading and Imageblocks, enabling tiled deferred rendering and MSAA sample coverage control in a single render pass with no intermediate main-memory reads or writes.

---
_Source: WWDC20 Session 10602 page (abstract, transcript, and resource links)._
