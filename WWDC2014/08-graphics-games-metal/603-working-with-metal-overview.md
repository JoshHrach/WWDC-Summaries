# Working with Metal: Overview
**WWDC14 · Session 603** · [Watch](https://developer.apple.com/videos/play/wwdc2014/603/)

_Platforms:_ iOS 8

## Overview
This session introduces Metal, Apple's brand-new low-overhead GPU programming API announced at WWDC 2014 and designed specifically for the A7 chip. Metal provides up to 10x the number of draw calls compared to OpenGL ES by fundamentally restructuring when expensive GPU programming operations (state validation, shader compilation, work submission) occur in the application lifecycle—moving them away from draw call time and into application build time and content-load time.

The session establishes the conceptual architecture of Metal: its objects (device, command queue, command buffer, command encoders, state objects, resources), its resource model (CPU/GPU unified memory, no implicit copies), its three command encoder types (Render, Compute, Blit), the Metal Shading Language (MSL), and the Xcode developer tools for debugging and profiling Metal applications. The session closes with a live Crytek demo of "The Collectables" using over 4,000 draw calls—a tenfold improvement over what was previously possible on iPad.

## Key Topics

### The Draw-Call Bottleneck Problem
Before Metal, each draw call required the CPU to perform state validation, translate API state to hardware state, and—often—recompile shaders. This CPU overhead limited practical draw call counts to a few hundred per frame. Metal eliminates this overhead by:
1. Moving shader compilation to **application build time** (via the Xcode Metal shader compiler).
2. Moving state validation and pipeline creation to **content load time** (via immutable state objects created from descriptors).
3. Reducing draw call time to just submitting pre-validated hardware commands to the GPU.

### Metal's Architecture: The Three Times
- **App build time**: `.metal` shader source files compiled by the Xcode Metal compiler into `.metallib` device-independent IR; errors and warnings surfaced at build time, not runtime.
- **Content load time**: Create `MTLRenderPipelineState`, `MTLDepthStencilState`, `MTLTexture`, `MTLBuffer`, `MTLSamplerState` — all expensive validation happens here.
- **Draw call time**: Only lightweight command encoding; no state validation, no shader recompilation.

### Key Metal Objects
- `MTLDevice` — abstraction of the GPU; root object of the entire API.
- `MTLCommandQueue` — ordered channel for submitting work to the GPU; one per application.
- `MTLCommandBuffer` — lightweight container of encoded hardware commands for one frame or sub-frame; many created per frame. Committed to the GPU explicitly.
- `MTLCommandEncoder` — translates API calls into hardware commands inside a command buffer. Three types: Render, Compute, Blit.
- State objects (`MTLRenderPipelineState`, `MTLDepthStencilState`, `MTLSamplerState`) — immutable compiled state; created from descriptors at load time.
- Resources (`MTLTexture`, `MTLBuffer`) — immutable structure, mutable contents; live on A7's unified CPU/GPU memory.

### Resource Model: Unified Memory
The A7's GPU and CPU share the same physical memory. Metal makes this explicit:
- No implicit copies of data between CPU and GPU.
- No cache flush API; Metal manages CPU/GPU cache coherency.
- Buffers: direct `void *` pointer access via `contents()`; no lock/unlock needed.
- Textures: implementation-private format optimized for GPU, with fast update routines. Can share underlying storage with buffers or other textures of the same pixel size for zero-copy reinterpretation.
- Resource structure is immutable after creation (dimensions, format, mip levels); contents are mutable.

### Command Encoders
- **MTLRenderCommandEncoder** — encodes one rendering pass (one framebuffer configuration). Contains all vertex/fragment pipeline state and draw calls. One encoder per set of render targets.
- **MTLComputeCommandEncoder** — encodes data-parallel GPU compute dispatch calls. Minimal state; same textures/buffers as graphics.
- **MTLBlitCommandEncoder** — encodes asynchronous data copy operations: texture uploads, buffer-to-buffer copies, mipmap generation. Runs in parallel with compute and graphics.

### Framebuffer Load and Store Actions
Metal exposes explicit tile-cache control for the A7's tile-based deferred renderer:
- `MTLLoadAction.clear` — clear the tile cache at the start of the pass (avoids loading previous contents from memory).
- `MTLLoadAction.load` — load the previous contents from memory.
- `MTLLoadAction.dontCare` — undefined starting state (fastest; safe when every pixel will be overwritten).
- `MTLStoreAction.store` — write tile cache to memory at the end of the pass.
- `MTLStoreAction.dontCare` — discard tile contents (saves memory bandwidth for intermediate buffers such as the depth buffer in a multi-pass renderer).
Setting these correctly can reduce memory traffic by 4x or more across a two-pass frame.

### Multi-Threading
- Multiple command buffers can be encoded in parallel across CPU threads.
- Execution order on the GPU is still controlled by the developer (commit order / dependency tracking).
- Implementation is lock-free; scales across CPU cores.

### Metal Shading Language (MSL)
- Based on C++11 (static subset); built on LLVM/clang.
- Unified: same language, toolchain, and tools for both graphics shaders and compute kernels.
- Extensions over C++: attribute syntax (`[[vertex_id]]`, `[[texture(n)]]`, `[[buffer(n)]]`), function overloading, basic template support, GPU-optimized standard library.
- Shaders are function arguments for all I/O; no global variables.
- Argument tables in command encoders map resource indices to shader function argument indices.

### Developer Tools
- **Xcode Metal Shader Compiler**: build-time compilation of `.metal` files; errors and warnings in the editor.
- **Metal Frame Debugger**: visual frame capture, Frame Navigator (left panel showing draw call order), Framebuffer View (center), Resource View (textures/buffers), Metal State Inspector (bottom).
- **Metal Shader Profiler**: per-shader cost breakdown, line-by-line timing within shader source.
- **Metal Performance Report**: frame rate, GPU utilization, most expensive shaders.

### Demo: Crytek "The Collectables"
Live demo on iPad demonstrating 4,000+ draw calls per frame using Metal's CRYENGINE integration—a 10x increase over OpenGL ES. Showcased Geom Cache technology (cache-based animation for complex destruction physics).

## APIs & Frameworks

**Metal** **[NEW]** — entire framework is new in iOS 8
- `MTLDevice` — `MTLCreateSystemDefaultDevice()`
- `MTLCommandQueue` — `MTLDevice.makeCommandQueue()`
- `MTLCommandBuffer` — `MTLCommandQueue.makeCommandBuffer()`
  - `commit()`, `addCompletedHandler(_:)`, `present(_:)`
- `MTLRenderCommandEncoder` — `commandBuffer.makeRenderCommandEncoder(descriptor:)`
  - `setRenderPipelineState(_:)`, `setVertexBuffer(_:offset:index:)`, `setFragmentTexture(_:index:)`, `setFragmentSamplerState(_:index:)`, `drawPrimitives(type:vertexStart:vertexCount:)`, `endEncoding()`
- `MTLComputeCommandEncoder` — `commandBuffer.makeComputeCommandEncoder()`
  - `setComputePipelineState(_:)`, `dispatchThreadgroups(_:threadsPerThreadgroup:)`, `endEncoding()`
- `MTLBlitCommandEncoder` — `commandBuffer.makeBlitCommandEncoder()`
  - `copy(from:to:)`, `generateMipmaps(for:)`, `endEncoding()`
- `MTLRenderPipelineDescriptor` — configures vertex/fragment functions, pixel formats, blend
- `MTLRenderPipelineState` — compiled immutable pipeline; `MTLDevice.makeRenderPipelineState(descriptor:)`
- `MTLDepthStencilDescriptor` / `MTLDepthStencilState`
- `MTLSamplerDescriptor` / `MTLSamplerState`
- `MTLTextureDescriptor` — `texture2DDescriptorWithPixelFormat:width:height:mipmapped:`
- `MTLTexture` — `MTLDevice.makeTexture(descriptor:)`
- `MTLBuffer` — `MTLDevice.makeBuffer(bytes:length:options:)`; `MTLBuffer.contents()` → `UnsafeMutableRawPointer`
- `MTLLibrary` — `MTLDevice.makeDefaultLibrary()`
- `MTLFunction` — `MTLLibrary.makeFunction(name:)`
- `MTLRenderPassDescriptor` — framebuffer configuration with `colorAttachments`, `depthAttachment`, `stencilAttachment`
- `MTLRenderPassColorAttachmentDescriptor` — `.texture`, `.loadAction`, `.storeAction`, `.clearColor`
- `MTLRenderPassDepthAttachmentDescriptor` — `.texture`, `.loadAction`, `.storeAction`, `.clearDepth`
- `MTLLoadAction` — `.clear`, `.load`, `.dontCare`
- `MTLStoreAction` — `.store`, `.dontCare`, `.multisampleResolve`
- `MTLPixelFormat` — `bgra8Unorm`, `depth32Float`, etc.
- `MTLPrimitiveType` — `.triangle`, `.line`, `.point`

**CAMetalLayer** (QuartzCore) **[NEW on iOS 8]**
- `CAMetalLayer` — `CALayer` subclass for Metal rendering
- `CAMetalLayer.nextDrawable()` → `CAMetalDrawable`
- `CAMetalDrawable.texture` → `MTLTexture`

**Metal Shading Language attributes** (all **[NEW]**)
- `[[vertex_id]]`, `[[instance_id]]`
- `[[stage_in]]`
- `[[position]]`, `[[point_size]]`, `[[clip_distance]]`, `[[front_facing]]`
- `[[buffer(n)]]`, `[[texture(n)]]`, `[[sampler(n)]]`
- `[[color(n)]]`, `[[depth(any)]]`, `[[sample_mask]]`
- `device`, `constant`, `threadgroup` address spaces
- `vertex`, `fragment`, `kernel` function qualifiers

**Xcode Tools (Metal-specific, all NEW)**
- Metal Shader Compiler (build-time)
- Metal Frame Debugger (runtime capture)
- Metal Performance Report / Shader Profiler

## Code Highlights

No sample code is presented in this overview session. See Session 604 (Fundamentals) and Session 605 (Advanced) for detailed code examples.

Key architectural pattern illustrated conceptually:
- Three objects for submission: `MTLDevice` → `MTLCommandQueue` → `MTLCommandBuffer` → commit.
- Two-phase state creation: `MTLRenderPipelineDescriptor` (mutable descriptor) → `MTLRenderPipelineState` (immutable compiled state).
- Argument table pattern: `encoder.setFragmentTexture(myTex, index: 1)` on the CPU side maps to `[[texture(1)]]` in the shader.

## Takeaways

- Metal eliminates the three primary sources of CPU overhead in GPU APIs (state validation, shader compilation, work submission) by moving them to earlier lifecycle stages, delivering up to 10x more draw calls per frame.
- The A7's unified CPU/GPU memory is exposed directly in Metal; developers access buffer contents with a raw pointer and take explicit responsibility for CPU/GPU synchronization instead of paying for implicit copies.
- Explicit `MTLLoadAction` and `MTLStoreAction` on framebuffer attachments are critical for performance on the A7's tile-based deferred renderer; discarding the depth buffer after a pass with `.dontCare` eliminates unnecessary memory writes.
- Metal is a unified graphics and compute API with a single shading language (MSL), enabling efficient interleaving of Render, Compute, and Blit operations within a single frame without expensive implicit transitions.

---
_Source: WWDC14 Session 603 page (abstract, chapter summaries, code samples, and resource links)._
