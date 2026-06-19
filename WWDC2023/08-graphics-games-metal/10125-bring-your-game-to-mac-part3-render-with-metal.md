# Bring Your Game to Mac, Part 3: Render with Metal
**WWDC23 · Session 10125** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10125/)

_Platforms:_ macOS Sonoma 14, macOS Ventura 13.3

## Overview
Part 3 of the "Bring Your Game to Mac" series closes out the porting journey by covering how to implement a native Metal renderer that takes full advantage of Apple Silicon's tile-based deferred rendering (TBDR) architecture. It walks through four key areas: managing GPU resource bindings and residency, optimizing command submission, translating indirect rendering (ExecuteIndirect), and integrating MetalFX upscaling.

The session is aimed at engineers who have already run their game under the Game Porting Toolkit (Part 1) and converted their HLSL shaders with Metal Shader Converter (Part 2). It provides Metal-idiomatic patterns for each major rendering subsystem, with practical Objective-C code and guidance on using the Metal Debugger in Xcode to find and fix bandwidth and performance issues.

## Key Topics

### Managing GPU Resources: Bindings and Residency
- Metal Argument Buffers map directly to Direct3D root signatures and descriptor tables. Two layout modes from Metal Shader Converter — Automatic and Explicit — control how HLSL register declarations translate to argument buffer fields.
- **Texture and sampler descriptor tables** are encoded as Metal buffers storing `MTLResourceID` values, built once outside the render loop. Setting `MTLSamplerDescriptor.supportArgumentBuffers = YES` is required for samplers used in argument buffers.
- A top-level argument buffer (the root signature equivalent) stores GPU addresses for each sub-table, enabling a single bind per shader at render time.
- **Residency strategy**: group all read-only resources into a `MTLHeap` (hazard tracking: Untracked) and call `useHeap` once per encoder; allocate writable resources individually and call `useResource` per encoder with explicit `MTLResourceUsage` flags — Metal handles synchronization automatically.

### Optimizing Command Submission
- Apple's TBDR GPU architecture uses fast on-chip tile memory. Keeping work within a render pass avoids costly system memory round-trips.
- Four best practices: (1) batch copy/blit commands before rendering; (2) group commands of the same type (render, compute, blit) into contiguous passes; (3) replace empty "clear" encoders with `MTLLoadActionClear` on the first encoder that uses a render target; (4) use `MTLStoreActionDontCare` for render targets not consumed by subsequent passes.
- In a demonstration example, optimizing load/store actions reduced five tile-memory/system-memory round-trips to one.
- **Metal Debugger in Xcode** surfaces these issues automatically in its Insights panel (Memory, Bandwidth, Performance, API Usage categories). The Dependencies viewer shows load/store actions and data flow between passes, making it easy to identify passes that can be merged.

### Indirect Rendering (ExecuteIndirect Translation)
- **Draw Indirect**: translate each ExecuteIndirect to a loop of `drawIndexedPrimitives:indirectBuffer:indirectBufferOffset:` calls — straightforward, works everywhere.
- **Indirect Command Buffers (ICBs)**: a superset of draw-indirect buffers that also encode PSO and buffer bindings from a GPU-side compute kernel, eliminating CPU encoding time for scenes with thousands of draw calls. Encode ICB commands using Metal Shading Language (`render_command` type); existing indirect-argument shaders can be reused unchanged by adding a small translation compute kernel between argument generation and the indirect rendering pass.
- `executeCommandsInBuffer:` schedules the ICB for GPU execution.

### MetalFX Upscaling
- MetalFX renders at a lower resolution and upscales to output resolution, saving frame time. Two algorithms: **Spatial** (best performance) and **Temporal** (near-native quality; requires jitter sequence and motion vectors, already present in engines with temporal AA).
- New in 2023: iOS support, up to 3× upscaling factor, Metal-cpp bindings.
- Temporal upscaling optionally accepts a 1×1 exposure texture or can use built-in autoexposure. Reset history on camera cuts and extreme camera movements.

## APIs & Frameworks

### Metal — Resource Binding
- `MTLResourceID` — GPU resource identifier stored in argument buffers
- `MTLBuffer.gpuAddress` — GPU virtual address for buffer encoding in argument buffers
- `MTLTexture.gpuResourceID` / `MTLSamplerState.gpuResourceID` — IDs for texture/sampler tables
- `MTLSamplerDescriptor.supportArgumentBuffers` — required `YES` for samplers in argument buffers
- `MTLDevice.newBufferWithLength:options:` — allocate argument buffer storage
- `MTLDevice.newTextureWithDescriptor:` / `MTLDevice.newSamplerStateWithDescriptor:` — resource creation

### Metal — Residency and Heaps
- `MTLHeapDescriptor` — describes heap size and type
- `MTLHeapType.automatic` — automatic sub-allocation
- `MTLDevice.newHeapWithDescriptor:` — create a heap
- `MTLHeap.newTextureWithDescriptor:` / `MTLHeap.newBufferWithLength:options:` — heap-backed allocation
- `MTLCommandEncoder.useHeap:` — make all heap resources resident for an encoder
- `MTLCommandEncoder.useResource:usage:stages:` — make individual resource resident with usage flags
- `MTLResourceUsage.read` / `MTLResourceUsage.write` — usage flags

### Metal — Render Pass Optimization
- `MTLLoadAction.clear` — clear render target at pass start (replaces empty encoder clears)
- `MTLLoadAction.load` — load existing content
- `MTLStoreAction.store` — store attachment to system memory
- `MTLStoreAction.dontCare` — discard attachment (saves bandwidth)
- `MTLRenderPassDescriptor` — configures load/store actions per attachment

### Metal — Indirect Rendering
- `MTLRenderCommandEncoder.drawIndexedPrimitives:indexType:indexBuffer:indexBufferOffset:indirectBuffer:indirectBufferOffset:` — draw indirect call
- `MTLDrawIndexedPrimitivesIndirectArguments` — structure of indirect draw arguments
- `MTLIndirectCommandBuffer` — **[NEW context]** GPU-encodable command buffer (ICB)
- `render_command` (MSL type) — encodes a render command inside a compute kernel for ICBs
- `render_command.set_render_pipeline_state` / `set_vertex_buffer` / `set_fragment_buffer` / `draw_indexed_primitives` — ICB encoding API in MSL
- `MTLRenderCommandEncoder.executeCommandsInBuffer:withRange:` — execute ICB
- `MTLIndirectCommandBufferDescriptor` — ICB configuration

### MetalFX
- `MTLFXSpatialScaler` — spatial upscaling (best performance)
- `MTLFXTemporalScaler` — temporal upscaling (near-native quality) **[extended: iOS, 3× scale, Metal-cpp NEW]**
- Temporal inputs: color texture, depth texture, motion vector texture, jitter offset, exposure texture or autoexposure
- `MTLFXTemporalScaler.reset` — reset history on camera cuts

### Xcode Tools
- Metal Debugger — GPU workload capture and analysis
- Insights panel — automatic detection of bandwidth, memory, performance, and API usage issues
- Dependencies viewer — visualizes load/store actions and data flow between render passes

## Code Highlights

Encoding a texture descriptor table as an argument buffer:
```objc
id<MTLBuffer> textureTable = [device newBufferWithLength:sizeof(MTLResourceID) * texturesCount
                                                  options:MTLResourceStorageModeShared];
MTLResourceID* ptr = (MTLResourceID*)textureTable.contents;
for (uint32_t i = 0; i < texturesCount; ++i) {
    id<MTLTexture> texture = [device newTextureWithDescriptor:textureDesc[i]];
    ptr[i] = texture.gpuResourceID;
}
```

Making heap resources resident (read-only resources):
```objc
// Create heap and allocate all read-only resources from it
id<MTLHeap> heap = [device newHeapWithDescriptor:heapDesc];
id<MTLTexture> texture = [heap newTextureWithDescriptor:desc];
// At render time — one call makes all resources resident
[encoder useHeap:heap];
```

Translating ExecuteIndirect to draw indirect calls:
```objc
for (uint32_t i = 0; i < maxDrawCount; ++i) {
    [renderEncoder drawIndexedPrimitives:MTLPrimitiveTypeTriangle
                               indexType:MTLIndexTypeUInt16
                             indexBuffer:indexBuffer
                       indexBufferOffset:indexBufferOffset
                          indirectBuffer:drawArgumentsBuffer
                    indirectBufferOffset:drawArgumentsBufferOffset];
    drawArgumentsBufferOffset += sizeof(MTLDrawIndexedPrimitivesIndirectArguments);
}
```

## Takeaways
- Group read-only resources in a `MTLHeap` and call `useHeap` once per encoder; let Metal manage synchronization for writable resources via `useResource` — this achieves bindless with minimal CPU overhead.
- Optimize render passes aggressively: batch copies up front, merge compatible passes, use `LoadActionClear` instead of empty encoder clears, and use `StoreActionDontCare` for transient attachments. Metal Debugger's Insights panel finds all of these automatically.
- Use draw-indirect for a quick ExecuteIndirect translation; switch to ICBs when CPU encoding time for thousands of draw calls becomes a bottleneck.
- MetalFX Temporal upscaling now supports iOS (new in 2023) and up to 3× scale — enable it whenever the engine already has TAA jitter and motion vectors.

---
_Source: WWDC23 Session 10125 page (abstract, chapter summaries, code samples, and resource links)._
