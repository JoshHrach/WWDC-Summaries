# Working with Metal: Advanced
**WWDC14 · Session 605** · [Watch](https://developer.apple.com/videos/play/wwdc2014/605/)

_Platforms:_ iOS 8

## Overview
This is the third session in the WWDC 2014 Metal trilogy, building on the fundamentals introduced in Sessions 603 and 604. It covers three major areas: structuring a full multi-pass rendering pipeline in Metal, data-parallel compute on the GPU using Metal compute kernels, and the Xcode developer tools for debugging and profiling Metal applications.

The rendering pipeline section uses a deferred lighting renderer with a shadow map pass and a G-buffer pass as a concrete vehicle for exploring descriptors, render pass descriptors, texture creation, render pipeline state objects, and command encoder sequencing. The compute section introduces Metal's data-parallel computing model (kernels, work-items, work-groups, local memory, synchronization barriers) and demonstrates image post-processing filters. The tools section demonstrates the Metal Frame Debugger and Shader Profiler live in Xcode, showing how to capture frames, inspect GPU command streams, debug framebuffer state, and find expensive shader hotspots.

## Key Topics

### Application Structure and Frequency of Operations
- **Once**: Create `MTLDevice`, create `MTLCommandQueue`.
- **As needed / level load**: Create off-screen textures (`MTLTextureDescriptor`), create render pass descriptors (`MTLRenderPassDescriptor`) for each known pass, load meshes and textures, compile shaders into `MTLLibrary`, create `MTLRenderPipelineState` and `MTLDepthStencilState` objects.
- **Every frame**: Acquire a new `MTLCommandBuffer`, update uniform buffers, call `addPresentedHandler`/`present`, commit command buffer.
- **Every render pass**: Create `MTLRenderCommandEncoder`, set states and resources, issue draw calls, call `endEncoding`.

### Descriptors as Blueprints
Descriptors define how Metal objects are created. Once the object is created, changes to the descriptor have no effect. Descriptors can be reused or modified slightly to create related objects.

### Multi-Pass Rendering: Framebuffer Configuration
- `MTLRenderPassDescriptor` — describes the framebuffer for a render pass: up to 4 color attachments, depth, and stencil.
- `MTLRenderPassAttachmentDescriptor` — specifies load action, store action, clear value, and texture for each attachment.
- Store actions: set intermediate G-buffer attachments to `MTLStoreActionDontCare` to save memory bandwidth; only the final displayable attachment uses `MTLStoreActionStore`.
- The drawable texture (from `CAMetalLayer`) must be assigned at frame time; all other off-screen textures are assigned at level-load time.

### Render Pipeline State Objects
The `MTLRenderPipelineState` bakes together: vertex function, fragment function, vertex layout (`MTLVertexDescriptor`), number of render targets, pixel formats, sample count, write masks, blend configuration, depth/stencil state. This eliminates runtime state validation and deferred shader recompilation. States that are cheap to change per draw call (cull mode, viewport, scissor, depth bias, buffer bindings) remain separate.

### Multi-Pass Encoding Flow
1. Create `MTLCommandBuffer` from `MTLCommandQueue`.
2. For each pass, create a `MTLRenderCommandEncoder` using the pre-built `MTLRenderPassDescriptor`.
3. Set pipeline state, bind buffers and textures, issue `drawIndexedPrimitives` calls.
4. Call `endEncoding`.
5. After all passes: call `commandBuffer.present(drawable)`, then `commandBuffer.commit`.
6. `MTLParallelRenderCommandEncoder` — alternative for parallelizing encoding across multiple CPU threads (noted, not detailed).

### Data-Parallel Computing with Metal Kernels
- **Kernel function**: a Metal Shading Language function qualified `kernel`, executed independently for each work-item. No graphics pipeline required.
- **Work-item**: one independent execution instance (indexed by global ID).
- **Work-group**: a set of work-items that can share data via local memory and synchronize with barriers.
- **Local memory**: user-managed high-bandwidth, low-latency memory, address space qualifier `threadgroup`.
- **Built-in attributes for kernels**: `[[thread_position_in_grid]]` (global ID), `[[thread_position_in_threadgroup]]` (local ID), `[[threadgroup_position_in_grid]]` (work-group ID), `[[thread_index_in_threadgroup]]` (local linear ID), `[[thread_position_in_grid]]` with 2D types for image processing.
- One work-item may operate on multiple pixels (e.g., 4 pixels per work-item) for better compute/memory ratio.

### Compute Command Encoder
1. Load kernel from `MTLLibrary`.
2. Create `MTLComputePipelineState` from the kernel function (triggers GPU binary compilation).
3. Create `MTLComputeCommandEncoder` from command buffer.
4. Set compute pipeline state and bind resources.
5. Call `dispatchThreadgroups(_:threadsPerThreadgroup:)` specifying work-group count and work-group size.
6. Call `endEncoding`.

### Xcode Metal Developer Tools
- **Pre-compiled shaders**: `.metal` files added to an Xcode project are compiled at build time; shader compiler errors and warnings appear in Xcode's editor at build time, not runtime.
- **Metal Frame Debugger**: triggered by the camera icon in the debug bar during a running Metal app. Captures all Metal commands, resources (buffers, textures, shaders) for the frame and allows replay to any command.
- **Debug Navigator (View Frame by Encoder)**: shows command encoders and commands in GPU execution order. Supports human-readable labels via `MTLObject.label`.
- **Push/Pop Debug Groups** (`pushDebugGroup(_:)`, `popDebugGroup()`): annotate command streams with named brackets visible in the navigator.
- **Variables View**: inspect the state of any Metal object at the selected command.
- **Back-trace from command**: jump directly to source where a `MTLRenderCommandEncoder` was created.
- **Shader Profiler (Sampling Profiler)**: available after frame capture; repeatedly runs shaders to gather samples; shows per-pipeline and per-draw-call cost; provides line-by-line profiling within the shader source editor.
- **Live shader editing**: modify a shader in the frame debugger, save, and immediately see recompiled results and updated profiling numbers without restarting the app.

## APIs & Frameworks

**Metal** **[NEW]**

_Device and Command Infrastructure_
- `MTLDevice` — represents the GPU; created with `MTLCreateSystemDefaultDevice()`
- `MTLCommandQueue` — channel to the GPU; created once
- `MTLCommandBuffer` — per-frame container for encoded commands; obtained from `MTLCommandQueue.makeCommandBuffer()`
- `commandBuffer.present(_:)` — schedule drawable presentation
- `commandBuffer.commit()` — submit to GPU

_Descriptors_
- `MTLTextureDescriptor` — defines texture dimensions, pixel format, mipmap count, texture type
- `MTLRenderPassDescriptor` — describes framebuffer for a render pass
- `MTLRenderPassColorAttachmentDescriptor` — load/store actions, clear color, texture for color attachment
- `MTLRenderPassDepthAttachmentDescriptor` — load/store actions, clear depth, texture for depth attachment
- `MTLRenderPassStencilAttachmentDescriptor` — load/store actions, clear stencil, texture for stencil attachment
- `MTLRenderPipelineDescriptor` — vertex/fragment functions, vertex layout, pixel formats, blend, sample count
- `MTLDepthStencilDescriptor` — depth/stencil comparison functions and write enables
- `MTLVertexDescriptor` — vertex attribute layout

_State Objects_
- `MTLRenderPipelineState` — immutable compiled render pipeline; created via `MTLDevice.makeRenderPipelineState(descriptor:)`
- `MTLDepthStencilState` — immutable depth/stencil state; created via `MTLDevice.makeDepthStencilState(descriptor:)`
- `MTLComputePipelineState` — immutable compiled compute pipeline; created via `MTLDevice.makeComputePipelineState(function:)`

_Resources_
- `MTLTexture` — created via `MTLDevice.makeTexture(descriptor:)`
- `MTLBuffer` — created via `MTLDevice.makeBuffer(length:options:)` or `makeBuffer(bytes:length:options:)`
- `MTLLibrary` — container for compiled Metal shader functions; obtained from `MTLDevice.makeDefaultLibrary()`
- `MTLFunction` — individual shader or kernel function from `MTLLibrary`

_Encoders_
- `MTLRenderCommandEncoder` — encodes render commands; created via `commandBuffer.makeRenderCommandEncoder(descriptor:)`
  - `setRenderPipelineState(_:)`
  - `setDepthStencilState(_:)`
  - `setVertexBuffer(_:offset:index:)`
  - `setFragmentTexture(_:index:)`
  - `setFragmentSamplerState(_:index:)`
  - `drawIndexedPrimitives(type:indexCount:indexType:indexBuffer:indexBufferOffset:)`
  - `pushDebugGroup(_:)` / `popDebugGroup()`
  - `endEncoding()`
- `MTLParallelRenderCommandEncoder` — parallel encoding across CPU threads (referenced)
- `MTLComputeCommandEncoder` — encodes compute commands; created via `commandBuffer.makeComputeCommandEncoder()`
  - `setComputePipelineState(_:)`
  - `setTexture(_:index:)`
  - `setBuffer(_:offset:index:)`
  - `dispatchThreadgroups(_:threadsPerThreadgroup:)`
  - `endEncoding()`
- `MTLBlitCommandEncoder` — referenced for GPU-side texture uploads

_Store/Load Action Constants_
- `MTLLoadAction` — `.clear`, `.load`, `.dontCare`
- `MTLStoreAction` — `.store`, `.dontCare`, `.multisampleResolve`

_CAMetalLayer_
- `CAMetalLayer.nextDrawable()` — obtains the drawable texture for display each frame
- `CAMetalDrawable.texture` — the `MTLTexture` to render into for display

**Metal Shading Language (MSL)**
- `kernel` qualifier — marks a function as a compute kernel
- `device` address space — GPU global memory (buffers)
- `threadgroup` address space — local memory shared within a work-group
- `[[thread_position_in_grid]]` — 1D or 2D global work-item ID
- `[[thread_position_in_threadgroup]]` — local work-item ID within the work-group
- `[[threadgroup_position_in_grid]]` — work-group ID
- `[[thread_index_in_threadgroup]]` — flat local index within work-group
- `threadgroup_barrier(mem_flags::)` — synchronization barrier within a work-group
- `texture2d<float, access::read>` / `texture2d<float, access::write>` — typed texture access in kernels

## Code Highlights

Creating a shadow map render pass descriptor at level-load time:
```objc
MTLTextureDescriptor *shadowTexDesc = [MTLTextureDescriptor
    texture2DDescriptorWithPixelFormat:MTLPixelFormatDepth32Float
    width:1024 height:1024 mipmapped:NO];
id<MTLTexture> shadowTex = [device newTextureWithDescriptor:shadowTexDesc];

MTLRenderPassDescriptor *shadowPassDesc = [MTLRenderPassDescriptor renderPassDescriptor];
shadowPassDesc.depthAttachment.texture = shadowTex;
shadowPassDesc.depthAttachment.loadAction = MTLLoadActionClear;
shadowPassDesc.depthAttachment.storeAction = MTLStoreActionStore;
shadowPassDesc.depthAttachment.clearDepth = 1.0;
```

Dispatching a compute kernel over a 2D image:
```objc
id<MTLComputeCommandEncoder> computeEncoder = [commandBuffer computeCommandEncoder];
[computeEncoder setComputePipelineState:postProcessState];
[computeEncoder setTexture:inputTexture atIndex:0];
[computeEncoder setTexture:outputTexture atIndex:1];

MTLSize threadgroupSize = MTLSizeMake(16, 16, 1);
MTLSize threadgroupCount = MTLSizeMake(
    (imageWidth  + 15) / 16,
    (imageHeight + 15) / 16, 1);
[computeEncoder dispatchThreadgroups:threadgroupCount
              threadsPerThreadgroup:threadgroupSize];
[computeEncoder endEncoding];
```

Metal Shading Language kernel with 2D global ID:
```metal
kernel void mirrorHorizontal(
    texture2d<float, access::read>  inTex  [[texture(0)]],
    texture2d<float, access::write> outTex [[texture(1)]],
    uint2 gid [[thread_position_in_grid]])
{
    uint2 mirrorCoord = uint2(inTex.get_width() - 1 - gid.x, gid.y);
    float4 color = inTex.read(mirrorCoord);
    outTex.write(color, gid);
}
```

## Takeaways

- Create all heavy Metal objects (textures, render pass descriptors, pipeline states) at level-load time; only command buffers and command encoders are created per frame.
- Setting G-buffer intermediate attachment store actions to `MTLStoreActionDontCare` is a critical performance optimization on tile-based GPUs.
- Metal's compute pipeline (`MTLComputeCommandEncoder` + `kernel` functions) provides a clean data-parallel model without a graphics pipeline; work-items may process multiple outputs each for better GPU occupancy.
- The Xcode Metal Frame Debugger with Shader Profiler provides line-by-line GPU hotspot identification and live shader editing, eliminating the need to restart the app during optimization.

---
_Source: WWDC14 Session 605 page (abstract, chapter summaries, code samples, and resource links)._
