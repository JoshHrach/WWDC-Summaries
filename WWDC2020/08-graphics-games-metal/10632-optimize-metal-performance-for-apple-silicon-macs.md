# Optimize Metal Performance for Apple silicon Macs
**WWDC20 · Session 10632** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10632/)

_Platforms:_ macOS Big Sur 11 (Apple silicon)

## Overview
This session teaches developers how to maximize GPU performance on the new Apple silicon Macs, which use Apple's Tile-Based Deferred Rendering (TBDR) GPU architecture — the same architecture found in iPhones and iPads. The session covers three main optimization areas: workload scheduling, minimizing system memory bandwidth, and maximizing hidden surface removal efficiency, followed by advanced optimizations using Metal's TBDR-specific features and shader core tuning.

The session is structured as two parts: Mike Imbrogno covers broad architectural optimizations (pass management, bandwidth reduction, HSR); Dom covers Metal TBDR-specific features (programmable blending, tile shaders, imageblock repurposing) and Apple GPU shader core optimizations (address spaces, data types, memory access patterns).

## Key Topics

### Workload Scheduling
- Apple GPUs overlap vertex, fragment, and compute stages concurrently using independent hardware channels; unnecessary dependencies serialize this work
- False dependencies arise from sharing Metal resources between adjacent passes even when the actual data is unrelated — fix by using separate resources or marking resources as **untracked**
- Metal fences and events allow explicit fine-grained synchronization when using untracked resources
- Always encode independent work early; reorder passes explicitly even though Metal can reorder automatically (its visibility is limited)
- Use **Metal System Trace** to visualize vertex/fragment/compute overlap and find scheduling bubbles

### Minimizing System Memory Bandwidth
- Load and store actions drive the majority of system bandwidth; minimize the number of render passes
- Replace multiple adjacent command buffers writing to the same attachments with `MTLParallelRenderCommandEncoder` (multi-threaded encoding without extra load/store cost)
- Merge adjacent passes when only load/store actions differ (e.g., opaque + translucent in a forward renderer)
- Use multiple render targets (up to 8 color attachments) to avoid attachment ping-ponging
- Fold clear operations into the render pass `loadAction = .clear` instead of using a standalone clear pass
- Resolve MSAA within the same render pass; mark MSAA textures as `memoryless` since sample data is rarely needed off-chip
- Mark transient attachments `storageMode = .memoryless` to eliminate memory footprint

### Hidden Surface Removal (HSR) Efficiency
- HSR eliminates overdraw for opaque fragments without extra passes; on Apple GPUs it replaces the need for a depth pre-pass
- Draw order: opaque → feedback (alpha test / discard / depth feedback) → translucent
- Use `[[early_fragment_tests]]` attribute on fragment functions that write to non-attachment resources so rejected fragments aren't shaded
- Avoid unintentional write masking — always write to all render pass attachments (initialize unused ones to zero) to prevent HSR from shading underlapping fragments
- Depth pre-passes are unnecessary when HSR is maximized; removing them eliminates z-fighting artifacts too

### Programmable Blending & On-Chip Deferred Rendering
- Programmable blending allows fragment shaders to read the current pixel's value from on-chip tile memory without touching system memory
- Use programmable blending to combine G-buffer generation and lighting into a single render pass; mark G-buffer attachments as `memoryless`
- Avoid `MTLBarrierScope.renderTargets` within a pass — this flushes tile memory to system memory on Apple GPUs

### Tile Shaders
- `MTLRenderCommandEncoder` supports tile shader dispatches (since A11); tile dispatches have implicit barriers against adjacent draw fragment stages
- Tile shaders access imageblocks and threadgroup memory; ideal for tile-based light culling inside a single render command encoder
- Configure `MTLRenderPassDescriptor.tileWidth/tileHeight` and `threadgroupMemoryLength` for tile shader workloads

### Imageblock Repurposing
- Use a fragment-based tile pipeline (tile shader with a fragment function) to transition imageblock layout mid-pass (e.g., G-buffer layout → multi-layer alpha blending layout)
- `[[imageblock_data]]` attribute marks imageblock input/output in tile shaders

### Apple GPU Shader Core Optimizations
- **Address spaces**: Use `constant` for data shared across all threads (enables compiler prefetch into uniform registers); use `device` for per-thread indexed data. Supply constants as a single struct by reference with compile-time-known array sizes.
- **16-bit data types**: Prefer `half` over `float` and `short` over `int`; 16-bit uses fewer registers, increases occupancy, and executes at higher ALU throughput. Use `h` suffix for half-precision literals (e.g., `2.0h`) to avoid implicit promotion to FP32.
- **Memory access patterns**: Avoid dynamically-indexed stack arrays (cause register spills); use signed index types to enable compiler vectorization; batch struct field accesses by placing adjacent fields together or using vector types.

## APIs & Frameworks

- **Metal** framework
- `MTLParallelRenderCommandEncoder` **[NEW advantage on Apple silicon]** — `makeParallelRenderCommandEncoder(descriptor:)`, `makeRenderCommandEncoder()`, `endEncoding()`
- `MTLRenderPassDescriptor` — `colorAttachments[n]`, `loadAction`, `storeAction`, `tileWidth`, `tileHeight`, `threadgroupMemoryLength`
- `MTLLoadAction` — `.clear`, `.load`, `.dontCare`
- `MTLStoreAction` — `.store`, `.dontCare`
- `MTLTexture` storage modes — `.memoryless` **[KEY for Apple GPU]**, `.private`, `.shared`
- `MTLTextureDescriptor` — `storageMode`
- `MTLRenderCommandEncoder` — tile shader dispatch support (A11+)
- `MTLFence` / `MTLEvent` — explicit synchronization primitives for untracked resources
- `MTLBarrierScope` — `.renderTargets` (avoid within a render pass on Apple GPUs)
- `MTLRenderPipelineDescriptor.colorAttachments[n].writeMask` — `MTLColorWriteMask`
- Metal Shading Language `[[early_fragment_tests]]` attribute — recover HSR efficiency for fragment functions with resource writes
- Metal Shading Language `[[imageblock_data]]` attribute — imageblock access in tile shaders
- Metal Shading Language address spaces: `device`, `constant`
- Metal Shading Language `half`, `short` types; `h` literal suffix
- `MTLComputePipelineState` — max threads per threadgroup, thread execution width
- **Metal System Trace** instrument — vertex/fragment/compute overlap visualization
- **Metal Debugger** (Xcode 12) — Pipeline Statistics, spill warnings

## Code Highlights

Parallel render command encoder for multi-threaded G-Buffer encoding:
```swift
let parallelEncoder = commandBuffer.makeParallelRenderCommandEncoder(descriptor: parallelDescriptor)
let subEncoder0 = parallelEncoder.makeRenderCommandEncoder()
let subEncoder1 = parallelEncoder.makeRenderCommandEncoder()
// async encode on worker threads, then:
syncPoint.wait()
parallelEncoder.endEncoding()
```

Multiple render target setup with memoryless attenuation attachment:
```swift
textureDescriptor.storageMode = .memoryless
let attenuationTexture = device.makeTexture(descriptor: textureDescriptor)
renderPassDesc.colorAttachments[1].texture    = attenuationTexture
renderPassDesc.colorAttachments[1].storeAction = .dontCare
```

Tiled deferred render pass setup with tile shaders:
```swift
renderPassDesc.tileWidth = 32
renderPassDesc.tileHeight = 32
renderPassDesc.threadgroupMemoryLength = MemoryLayout<LightInfo>.size * 8
```

Imageblock layout transition (G-buffer → multi-layer alpha blending):
```metal
fragment FragmentOutput my_tile_shader(DeferredShadingFragment input [[imageblock_data]]) {
    FragmentOutput output;
    output.v.color_and_transmittence[0] = half4(input.lighting, 0.0h);
    output.v.depth[0] = input.depth;
    return output;
}
```

## Takeaways

- Replace multiple adjacent command buffers with `MTLParallelRenderCommandEncoder` and mark transient attachments `.memoryless` — these two changes alone can dramatically reduce system memory bandwidth.
- Always eliminate false scheduling dependencies and reorder independent passes early; use Metal System Trace to verify vertex/fragment/compute overlap.
- On Apple GPUs, a depth pre-pass is unnecessary — maximize HSR by drawing opaque geometry first, writing to all attachments, and using `[[early_fragment_tests]]` where appropriate.
- Prefer 16-bit types (`half`, `short`) in shaders, use the `constant` address space for shared data with compile-time-known sizes, and use signed indices to enable compiler vectorization.

---
_Source: WWDC20 Session 10632 page (abstract, chapter summaries, code samples, and resource links)._
