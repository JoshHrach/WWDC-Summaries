# Modern Rendering with Metal
**WWDC19 · Session 601** · [Watch](https://developer.apple.com/videos/play/wwdc2019/601/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, tvOS 13

## Overview
This session covers three interconnected areas of modern Metal rendering: a survey of advanced rendering techniques (deferred, tiled deferred, tiled forward, and visibility buffer), a deep dive into GPU-driven rendering pipelines using argument buffers and indirect command buffers, and a new GPU Family API that unifies feature querying across Apple platforms.

The rendering techniques section walks through classic two-pass deferred rendering, then shows how iOS hardware's tiled architecture enables programmable blending (merging geometry and lighting passes into a single render encoder with G-buffer kept entirely in tile memory), tile shaders with persistent thread group memory for tiled deferred lighting, and clustered lighting for depth-prepass-free forward rendering. A new Metal 3 addition — barycentric coordinates and primitive indices in fragment shaders — makes visibility buffer rendering far easier to implement and broadens its viability on Mac.

The GPU-driven pipeline section demonstrates moving frustum culling, occlusion culling, and LOD selection entirely to the GPU using compute kernels that encode draw calls into indirect command buffers. Metal 3 extends this with compute dispatches in indirect command buffers, enabling per-patch tessellation factor generation on the GPU. A demo of a 2.8-million-polygon bistro scene with ~8,000 draw calls and four shadow cascade views shows the practical gains.

## Key Topics
- **Deferred rendering** — two-pass setup (geometry pass writes G-buffer; lighting pass reads it); `MTLRenderPassDescriptor` load/store action patterns (`dontcare` vs `store`)
- **Programmable blending (iOS)** — merge geometry and lighting into a single `MTLRenderCommandEncoder`; G-buffer attachments remain in tile memory; `MTLTextureDescriptor.storageMode = .memoryless` eliminates physical G-buffer allocation
- **Tiled deferred lighting** — compute prepass fits light subfrustums to depth bounds, builds per-tile light lists; on iOS, tile shaders (`MTLTileRenderPipelineDescriptor`) with persistent thread group memory run the prepass in-encoder, storing light lists in tile memory
- **Tiled / clustered forward rendering** — depth prepass or clustered 3D light frustum subdivision for depth-prepass-free variant; reuses same tile light list for forward draws
- **Visibility buffer** — stores only primitive ID and barycentric coordinates in G-buffer; reconstructs geometry in lighting pass; now easier than ever with new Metal 3 fragment shader attributes
- **GPU-driven pipelines** — argument buffers encode full scene data (meshes, materials, models, LOD arrays) as nested structures; compute kernels do frustum/occlusion culling and LOD selection per-thread; results encoded into indirect command buffers (`ICB`)
- **Indirect ranges** — atomically increment a length counter in a range buffer as commands are packed; `executeCommandsInBuffer(_:indirectBuffer:)` executes only the valid range, avoiding empty-slot overhead
- **GPU-driven compute dispatches** — encode compute dispatches into ICBs on the GPU; per-object culling kernel can encode per-patch tessellation compute dispatches, fully paralyzing the tessellation factor pass **[NEW Metal 3]**
- **GPU Family API** — four families (Apple, Mac, Common, iOS Mac) replace the proliferating feature set enumerations; hierarchical instances; `MTLDevice.supportsFamily(_:)` replaces `supportsFeatureSet(_:)` **[NEW]**

## APIs & Frameworks
- **Metal**
  - `MTLRenderPassDescriptor` — `colorAttachments[n].loadAction` / `storeAction` (`dontcare`, `store`, `clear`)
  - `MTLTextureDescriptor.storageMode = .memoryless` **[NEW usage pattern for iOS]** — G-buffer textures with no physical backing
  - Programmable blending — reading color attachments of the same pixel in a fragment shader (iOS hardware tile architecture)
  - `MTLTileRenderPipelineDescriptor` — tile shader pipeline state for in-encoder compute on iOS
  - `threadgroupMemoryLength` on render pass descriptor — reserve persistent thread group memory for tile light lists
  - `MTLRenderCommandEncoder.setTileBytes(_:length:index:)` / `dispatchThreadsPerTile(_:)` — encode tile shader dispatch
  - `[[barycentric_coord]]` fragment shader attribute **[NEW Metal 3]** — barycentric coordinates per pixel
  - `[[primitive_id]]` fragment shader attribute **[NEW Metal 3]** — primitive index for visibility buffer technique
  - `MTLArgumentEncoder` / argument buffers — encode structured scene data (meshes, materials, model LOD arrays) for GPU access
  - `MTLIndirectCommandBuffer` — encode draw commands on the GPU; built once, reused
    - `MTLIndirectRenderCommand` — set pipeline state, vertex/fragment buffers, encode draw
    - `MTLIndirectCommandBuffer` for compute dispatches **[NEW Metal 3]**
  - `MTLRenderCommandEncoder.executeCommandsInBuffer(_:range:)` — execute ICB commands
  - `MTLRenderCommandEncoder.executeCommandsInBuffer(_:indirectBuffer:indirectBufferOffset:)` — indirect range variant **[NEW]**
  - `MTLDevice.supportsFamily(_:)` **[NEW]** — query GPU family membership
  - `MTLGPUFamily` enum — `.apple1`–`.apple5`, `.mac1`–`.mac2`, `.common1`–`.common3`, `.iOSMac1`–`.iOSMac2` **[NEW]**
  - `MTLDevice.supportsFeatureSet(_:)` — legacy API (still supported)
  - `MTLDevice.location`, `.locationNumber`, `.maxTransferRate` — multi-GPU topology queries **[NEW]**
  - `MTLSharedEvent` — cross-GPU synchronization via `encodeSignalEvent` / `encodeWaitForEvent`
- **Platform-specific new features**
  - iOS / tvOS: pipeline state objects in ICBs, indirect ranges, 16-bit depth texture support **[NEW]**
  - macOS: renderless render passes (no attachments), GPU command buffer execution time query, sRGB/non-sRGB view casting **[NEW]**

## Code Highlights

```metal
// Visibility buffer: barycentric coordinates and primitive ID in fragment shader (Metal 3)
fragment float4 lightingShader(
    VertexOut in [[stage_in]],
    float3 baryCoord [[barycentric_coord]],   // NEW Metal 3
    uint primitiveID [[primitive_id]],         // NEW Metal 3
    constant SceneData &scene [[buffer(0)]])
{
    // Reconstruct geometry from primitive ID + barycentric coords
    Mesh mesh = scene.meshes[primitiveID];
    float3 worldPos = interpolate(mesh, baryCoord);
    // ... full material evaluation here
}
```

```swift
// GPU-driven culling kernel encodes draw calls into ICB
// (conceptual setup code)
let icbDescriptor = MTLIndirectCommandBufferDescriptor()
icbDescriptor.commandTypes = .draw
icbDescriptor.inheritPipelineState = false
let icb = device.makeIndirectCommandBuffer(descriptor: icbDescriptor, maxCommandCount: objectCount)!

// Range buffer for atomic packing
var rangeBuffer = device.makeBuffer(length: MemoryLayout<MTLIndirectCommandBufferExecutionRange>.size)!

// Compute pass: culling kernel reads scene argument buffer, encodes visible draws
computeEncoder.setBuffer(sceneArgumentBuffer, offset: 0, index: 0)
computeEncoder.setBuffer(rangeBuffer, offset: 0, index: 1)
computeEncoder.useResource(icb, usage: .write)
computeEncoder.dispatchThreads(MTLSize(width: objectCount, height: 1, depth: 1),
                                threadsPerThreadgroup: MTLSize(width: 64, height: 1, depth: 1))

// Render pass: execute only packed commands via indirect range
renderEncoder.executeCommandsInBuffer(icb, indirectBuffer: rangeBuffer, indirectBufferOffset: 0)
```

```swift
// New GPU family query replaces feature set checks
if device.supportsFamily(.apple4) {
    // Tile shaders, persistent thread group memory, programmable blending
} else if device.supportsFamily(.common2) {
    // Indirect draws, counting occlusion queries, tessellation
}
```

## Takeaways
- Programmable blending on iOS completely eliminates G-buffer memory and bandwidth overhead — set attachment storage mode to `.memoryless` and merge geometry/lighting into a single encoder; this is the highest-leverage iOS rendering optimization available.
- The new `[[barycentric_coord]]` and `[[primitive_id]]` fragment shader attributes in Metal 3 make visibility buffer rendering practical across all Apple Silicon platforms, not just tiled GPUs.
- GPU-driven pipelines with argument buffers and indirect command buffers should be the default architecture for scenes with thousands of objects — frustum culling, occlusion culling, LOD selection, and now tessellation factor generation all run in massively parallel compute kernels without any CPU-GPU synchronization.
- Replace all `supportsFeatureSet(_:)` calls with `supportsFamily(_:)` using the new `MTLGPUFamily` hierarchy — four families (Apple, Mac, Common, iOS Mac) cover all conditional feature branches cleanly across iOS and macOS.

---
_Source: WWDC19 Session 601 page (full transcript, abstract, and resource links)._
