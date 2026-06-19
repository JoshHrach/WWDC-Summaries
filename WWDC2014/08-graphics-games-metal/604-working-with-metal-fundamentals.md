# Working with Metal: Fundamentals
**WWDC14 · Session 604** · [Watch](https://developer.apple.com/videos/play/wwdc2014/604/)

_Platforms:_ iOS 8

## Overview
This is the second session in the WWDC 2014 Metal trilogy, turning the architectural concepts introduced in Session 603 into concrete code. The first half walks step by step through building a "hello world" Metal application that renders a single animated triangle, covering device and command queue setup, buffer creation, render pipeline state construction, CAMetalLayer integration, and GPU–CPU synchronization using dispatch semaphores. The second half provides a comprehensive tour of the Metal Shading Language (MSL), covering scalar/vector/matrix types, address spaces, vertex and fragment shader inputs/outputs, vertex descriptor–based attribute matching, shader pairing rules, math modes, and the Metal standard library.

Together, the two halves give developers everything they need to write their first real Metal application and understand how the shading language maps to the GPU execution model.

## Key Topics

### Building a Metal Application: Initialization
1. **Get the device** — `MTLCreateSystemDefaultDevice()` returns the single GPU device.
2. **Create a command queue** — one per application, used to submit work to the GPU.
3. **Create resources** — `MTLBuffer` for vertex data; CPU/GPU shared memory, no lock/unlock needed.
4. **Create the render pipeline** — `MTLRenderPipelineDescriptor` sets vertex function, fragment function, and framebuffer pixel format. `MTLDevice.makeRenderPipelineState(descriptor:)` performs offline shader compilation and returns an immutable `MTLRenderPipelineState`.
5. **Create the view** — `UIView` subclass overriding `layerClass` to return `CAMetalLayer`; the layer manages a swap chain of (typically 3) display textures.

### Rendering a Frame
1. `CAMetalLayer.nextDrawable()` — gets the next available texture from the swap chain (may block if all are in use).
2. Configure `MTLRenderPassDescriptor` with the drawable's texture as color attachment 0; set load action to `.clear`.
3. `commandBuffer.makeRenderCommandEncoder(descriptor:)` — begin the render pass.
4. Set pipeline state and vertex buffer, call `drawPrimitives(type:vertexStart:vertexCount:)`.
5. `encoder.endEncoding()`.
6. `commandBuffer.present(drawable)` — schedule display presentation via CoreAnimation callback.
7. `commandBuffer.commit()` — enqueue to GPU for execution.

### GPU–CPU Synchronization with Uniform Buffers
Writing to a shared uniform buffer while the GPU is still reading it creates a race condition. The solution is a pool of N uniform buffers (typically 3, matching the swap chain depth) protected by a `DispatchSemaphore(value: N)`:
- **Wait** (semaphore) at the top of the loop before writing to the next buffer.
- **Signal** the semaphore inside the `commandBuffer.addCompletedHandler` block when the GPU finishes.
This same pattern applies to dynamic vertex data and streaming texture updates.

### Metal Shading Language (MSL) Overview
MSL is based on C++11. It does not use the C++ standard library; it provides its own GPU-optimized standard library. Functions use the `metal` namespace.

**Shader qualifiers:**
- `vertex` — marks a vertex shader function
- `fragment` — marks a fragment shader function
- `kernel` — marks a compute kernel (covered in Session 605)

**All inputs are function arguments; all outputs are return values.** There are no global variables for I/O.

### MSL Data Types
- **Scalars**: C++11 types plus `half` (IEEE 754 16-bit float) — prefer `half` for performance and power efficiency.
- **Vectors**: `float2`, `float4`, `half2`, `half4`, etc. Aligned to vector-length bytes. Use `packed_float3`, `packed_half4`, etc. to avoid compiler-generated padding when building tightly packed vertex structures shared with host code.
- **Matrices**: floating-point matrices in column-major order; match SIMD types from iOS 8's Accelerate/SIMD library, enabling zero-copy struct sharing between host code and shaders.
- **Atomics**: subset of C++11 atomic types; used for race-free read-modify-write operations (e.g., histogram generation).
- **Textures**: declared as templates — `texture2d<float, access::sample>`, `texture2d<float, access::write>`, `depth2d<float>`. Access modes: `sample`, `read`, `write`. Cannot mix `sample` and `write` on the same texture in one shader.
- **Samplers**: passed as arguments or declared inline using variadic template syntax; default values cover most cases.
- **Buffers**: pointers or references qualified with an address space.

### Address Spaces
- `device` — GPU global memory; use when each thread/instance accesses a unique index (vertex data, per-instance data).
- `constant` — read-only data accessed identically by all shader instances (uniform matrices, light descriptors, skinning matrices); pass by reference so the compiler can prefetch for a significant performance gain.
- `threadgroup` — shared within a compute work-group (covered in Session 605).

### Vertex Shader Inputs
Two approaches:
1. **Direct buffer indexing** — pass a pointer to a struct in `device` address space; index using `[[vertex_id]]` or `[[instance_id]]`.
2. **Vertex descriptor** — declare inputs as a struct with `[[stage_in]]` qualifier; use `MTLVertexDescriptor` at runtime to specify buffer indices, attribute offsets, and formats. Decouples shader layout from host data layout.

### Vertex/Fragment Shader Pairing
- Vertex output struct elements must match fragment input struct elements by attribute name and type.
- Built-in attributes: `[[position]]` (must always be returned), `[[point_size]]`, `[[clip_distance]]`, `[[front_facing]]`, `[[color(n)]]`, `[[depth(any)]]`, `[[sample_mask]]`.
- User-defined attributes use `[[user(name)]]`; the fragment input may be a subset of the vertex output.
- Order of elements in the structs does not have to match.

### Math Modes
- Default: **fast math** — optimized for performance; NaN behavior undefined; trig functions have limited range.
- **Precise math** — IEEE-compliant; invoke via `metal::precise::sin(x)` etc. for specific calls, or set compiler option for the entire shader.

## APIs & Frameworks

**Metal** **[NEW]**
- `MTLDevice` — root GPU object; `MTLCreateSystemDefaultDevice()` **[NEW]**
- `MTLCommandQueue` — GPU submission channel; `MTLDevice.makeCommandQueue()` **[NEW]**
- `MTLCommandBuffer` — per-frame command container; `MTLCommandQueue.makeCommandBuffer()` **[NEW]**
  - `addCompletedHandler(_:)` — GPU completion callback
  - `present(_:)` — schedule drawable presentation
  - `commit()` — submit to GPU
- `MTLBuffer` — raw CPU/GPU shared memory buffer **[NEW]**
  - `MTLDevice.makeBuffer(bytes:length:options:)`
  - `MTLBuffer.contents()` → `UnsafeMutableRawPointer` — direct pointer to shared memory
- `MTLTexture` — GPU texture resource **[NEW]**
- `MTLLibrary` — compiled shader container; `MTLDevice.makeDefaultLibrary()` **[NEW]**
- `MTLFunction` — individual shader function; `MTLLibrary.makeFunction(name:)` **[NEW]**
- `MTLRenderPipelineDescriptor` — pipeline configuration **[NEW]**
  - `.vertexFunction`, `.fragmentFunction`
  - `.colorAttachments[0].pixelFormat`
  - `.vertexDescriptor` → `MTLVertexDescriptor`
- `MTLRenderPipelineState` — compiled, immutable pipeline object **[NEW]**
  - `MTLDevice.makeRenderPipelineState(descriptor:)`
- `MTLVertexDescriptor` — vertex attribute layout descriptor **[NEW]**
  - `MTLVertexAttributeDescriptor` — format, offset, buffer index
  - `MTLVertexBufferLayoutDescriptor` — stride, step function
- `MTLRenderPassDescriptor` — framebuffer configuration **[NEW]**
  - `colorAttachments[0].texture`, `.loadAction`, `.clearColor`
- `MTLRenderCommandEncoder` — render command encoding **[NEW]**
  - `setRenderPipelineState(_:)`
  - `setVertexBuffer(_:offset:index:)`
  - `drawPrimitives(type:vertexStart:vertexCount:)`
  - `endEncoding()`
- `MTLDepthStencilDescriptor` / `MTLDepthStencilState` **[NEW]**
- `MTLLoadAction` — `.clear`, `.load`, `.dontCare` **[NEW]**
- `MTLStoreAction` — `.store`, `.dontCare` **[NEW]**

**CAMetalLayer** (QuartzCore) **[NEW on iOS 8]**
- `CAMetalLayer` — `CALayer` subclass that vends displayable Metal textures
- `CAMetalLayer.nextDrawable()` → `CAMetalDrawable` — acquires next texture from swap chain (may block)
- `CAMetalDrawable.texture` — `MTLTexture` to render into

**UIKit**
- `UIView.layerClass` — override to return `CAMetalLayer.self`

**Swift/SIMD (iOS 8)**
- `simd` module — vector/matrix types (`float2`, `float4x4`, etc.) shared between host code and MSL shaders without conversion

**Metal Shading Language qualifiers and attributes** (all **[NEW]**)
- `vertex`, `fragment`, `kernel` — function qualifiers
- `[[vertex_id]]`, `[[instance_id]]` — per-vertex/instance IDs
- `[[stage_in]]` — per-vertex or per-fragment user-defined input struct
- `[[position]]` — clip-space position output (required from vertex shader)
- `[[point_size]]`, `[[clip_distance]]` — built-in vertex outputs
- `[[front_facing]]` — fragment built-in input
- `[[color(n)]]` — color attachment index for fragment output
- `[[depth(any)]]` — fragment depth output
- `[[user(name)]]` — user-defined attribute name for vertex↔fragment pairing
- `[[buffer(n)]]`, `[[texture(n)]]`, `[[sampler(n)]]` — resource binding indices
- `device`, `constant`, `threadgroup` — address space qualifiers
- `metal::fast::`, `metal::precise::` — math mode namespaces

## Code Highlights

Five-step Metal initialization:
```objc
// 1. Device
id<MTLDevice> device = MTLCreateSystemDefaultDevice();
// 2. Command queue
id<MTLCommandQueue> commandQueue = [device newCommandQueue];
// 3. Vertex buffer (CPU/GPU shared memory)
id<MTLBuffer> vertexBuffer = [device newBufferWithBytes:vertexData
                                                 length:sizeof(vertexData)
                                                options:MTLResourceStorageModeShared];
// 4. Render pipeline
MTLRenderPipelineDescriptor *desc = [[MTLRenderPipelineDescriptor alloc] init];
id<MTLLibrary> lib = [device newDefaultLibrary];
desc.vertexFunction   = [lib newFunctionWithName:@"vertexShader"];
desc.fragmentFunction = [lib newFunctionWithName:@"fragmentShader"];
desc.colorAttachments[0].pixelFormat = MTLPixelFormatBGRA8Unorm;
id<MTLRenderPipelineState> pipelineState = [device newRenderPipelineStateWithDescriptor:desc error:nil];
// 5. View / CAMetalLayer configured in UIView subclass
```

Uniform buffer triple-buffering with semaphore:
```objc
dispatch_semaphore_t sem = dispatch_semaphore_create(3);
// Per frame:
dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
// Write to uniformBuffers[currentBufferIndex]
id<MTLCommandBuffer> cmd = [commandQueue commandBuffer];
[cmd addCompletedHandler:^(id<MTLCommandBuffer> _) {
    dispatch_semaphore_signal(sem);
}];
[cmd commit];
```

Pass-through vertex shader (MSL):
```metal
#include <metal_stdlib>
using namespace metal;

struct VertexIn  { float4 position; float4 color; };
struct VertexOut { float4 position [[position]]; float4 color; };

vertex VertexOut vertexShader(
    device const VertexIn *vertices [[buffer(0)]],
    constant     Uniforms &uniforms [[buffer(1)]],
    uint          vid               [[vertex_id]])
{
    VertexOut out;
    out.position = uniforms.mvp * vertices[vid].position;
    out.color    = vertices[vid].color;
    return out;
}

fragment float4 fragmentShader(VertexOut in [[stage_in]]) {
    return in.color;
}
```

## Takeaways

- Every Metal resource (buffer, texture, pipeline state) is created on `MTLDevice`; there is no concept of a context or binding group as in OpenGL.
- `MTLRenderPipelineState` is the central object that bakes vertex/fragment shaders together with framebuffer configuration, eliminating runtime state validation and deferred compilation hitches.
- Shared CPU/GPU `MTLBuffer` memory is directly accessible from both the CPU (`contents()`) and GPU; synchronize access with dispatch semaphores and `addCompletedHandler` to avoid race conditions.
- The Metal Shading Language shares scalar and vector types with iOS 8's SIMD library, allowing struct definitions to be shared between host Objective-C/Swift code and shaders via a common header.

---
_Source: WWDC14 Session 604 page (abstract, chapter summaries, code samples, and resource links)._
