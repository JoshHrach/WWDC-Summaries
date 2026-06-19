# Bringing OpenGL Apps to Metal
**WWDC19 · Session 611** · [Watch](https://developer.apple.com/videos/play/wwdc2019/611/)

_Platforms:_ iOS 13, iPadOS 13, macOS Catalina 10.15, tvOS 13

## Overview
OpenGL, OpenGL ES, and OpenCL are deprecated as of iOS 12 / macOS Mojave and will continue to work in iOS 13 and macOS Catalina, but new projects should target Metal from day one. This session provides a step-by-step migration guide for developers who have existing OpenGL renderers, walking through every stage of the typical graphics app lifecycle and showing the direct Metal equivalent for each concept.

The session demonstrates that porting is not a complete engine rewrite — the fundamental flow (create window → create resources → compile shaders → setup state → render loop → present) is preserved. Metal's explicit design eliminates the hidden validation and unpredictable submission timing that make OpenGL performance difficult to reason about, moving expensive work (shader compilation, state validation) earlier in the app lifecycle.

A live demo with Xcode's frame capture and Metal Shader Debugger shows how to spot and fix a common texture y-coordinate flip error that arises during porting.

## Key Topics

**Window / View System**
- `NSOpenGLView` / `GLKView` → `MTKView`; `EAGLLayer` → `CAMetalLayer`
- `MTKViewDelegate` provides separate `mtkView(_:drawableSizeWillChange:)` callback; no need to check resolution inside `draw(in:)`
- `CAMetalLayer` exposes `nextDrawable()` to obtain textures from a managed pool

**Command Submission (OpenGL implicit → Metal explicit)**
- `MTLDevice` — abstract GPU representation; replaces glContext
- `MTLCommandQueue` — ordered submission pipeline; allocated once
- `MTLCommandBuffer` — filled list of GPU commands; committed explicitly
- Encoders: `MTLBlitCommandEncoder` (copy), `MTLComputeCommandEncoder` (compute), `MTLRenderCommandEncoder` (draw)
- Replace `glFlush`/`glFinish` with `waitUntilScheduled()`/`waitUntilCompleted()` (sparingly) or completion handlers

**Resource Creation**
- `MTLBuffer` and `MTLTexture` containers are immutable after creation; contents are mutable
- Storage modes replace GL usage hints: `.shared` (CPU+GPU, unified memory), `.private` (GPU only, fast), `.managed` (macOS: separate copies, explicit sync via `didModifyRange`)
- Texture coordinates are flipped on y-axis vs. GL; fix in texture loading, not shaders
- Metal performs no pixel format conversion; upload textures in their intended format

**Shader Compilation**
- GLSL → Metal Shading Language (MSL); MSL is C++ based
- Shaders compiled at Xcode build time into default Metal library (`.metallib`) in app bundle; no runtime GLSL compilation
- MSL entry points have explicit stage qualifiers (`vertex`, `fragment`, `kernel`) and distinct names
- Uniform bindings are explicit slot indices in MSL (not string-named); typed via `constant` keyword
- Built-in SIMD types: `float2`, `float3`, `float4` etc. (via `simd.h`); aligned to 16 bytes

**Render State Objects**
- `MTLRenderPipelineDescriptor` / `MTLRenderPipelineState` — combines vertex/fragment functions, vertex layout, pixel formats, blend state (replaces GL program + VAO + FBO attachment format)
- `MTLDepthStencilDescriptor` / `MTLDepthStencilState` — prebaked depth/stencil state
- PSOs created once, validated at creation time; swap between draw calls instead of mutating state
- Replaces GL shader pre-warming with explicit PSO pre-creation

**Render Loop & Resource Updates**
- `MTLRenderPassDescriptor` with explicit load/store actions (`.clear`, `.store`, `.dontCare`) replaces FBO bind + glClear + glDiscardFramebufferEXT
- `MTLBlitCommandEncoder` for GPU-side buffer/texture copies
- Triple-buffering pattern using a semaphore (`DispatchSemaphore`) and rotating buffer pool prevents CPU/GPU race conditions
- Avoid `waitUntilCompleted` in render loop; use completion handlers instead

**Presenting**
- Call `commandBuffer.present(drawable)` before `commit()` instead of relying on context `presentRenderbuffer`

**Tooling (New in Xcode 11 / macOS Catalina)**
- Metal Shader Debugger — inspect per-line values for a selected pixel
- GPU Frame Capture — record every Metal API call for post-mortem analysis
- GPU Memory Viewer — visualize and optimize memory usage **[NEW]**
- Metal in Simulator — full hardware-accelerated Metal support for iOS/tvOS Simulator using `MTLGPUFamilyApple2` feature set **[NEW]**
- Instruments Game Performance template with Metal System Trace

## APIs & Frameworks

### MetalKit
- `MTKView` — Metal-backed view with `MTKViewDelegate`
- `MTKViewDelegate.mtkView(_:drawableSizeWillChange:)` — resize callback
- `MTKViewDelegate.draw(in:)` — render callback

### Metal
- `MTLDevice` — GPU abstraction; `MTLCreateSystemDefaultDevice()`
- `MTLCommandQueue` — `device.makeCommandQueue()`
- `MTLCommandBuffer` — `queue.makeCommandBuffer()`
- `MTLBlitCommandEncoder` — `commandBuffer.makeBlitCommandEncoder()`
- `MTLComputeCommandEncoder` — `commandBuffer.makeComputeCommandEncoder()`
- `MTLRenderCommandEncoder` — `commandBuffer.makeRenderCommandEncoder(descriptor:)`
- `MTLBuffer` — `device.makeBuffer(length:options:)`; `StorageMode`: `.shared`, `.private`, `.managed`
- `MTLTexture` / `MTLTextureDescriptor` — `device.makeTexture(descriptor:)`; `replaceRegion(_:mipmapLevel:withBytes:bytesPerRow:)`
- `MTLSamplerState` / `MTLSamplerDescriptor` — `device.makeSamplerState(descriptor:)`
- `MTLLibrary` — `device.makeDefaultLibrary()`; `library.makeFunction(name:)`
- `MTLRenderPipelineDescriptor` / `MTLRenderPipelineState` — `device.makeRenderPipelineState(descriptor:)`
- `MTLDepthStencilDescriptor` / `MTLDepthStencilState` — `device.makeDepthStencilState(descriptor:)`
- `MTLRenderPassDescriptor` — load/store actions per attachment
- `MTLRenderPassColorAttachmentDescriptor.loadAction` (`.clear`, `.load`, `.dontCare`)
- `MTLRenderPassColorAttachmentDescriptor.storeAction` (`.store`, `.dontCare`)
- `MTLCommandBuffer.present(_:)` — present drawable
- `MTLCommandBuffer.addCompletedHandler(_:)` — async GPU completion callback
- `MTLCommandBuffer.waitUntilCompleted()` — synchronous wait (avoid in render loop)
- `MTLBuffer.didModifyRange(_:)` — sync managed buffer CPU→GPU (macOS)
- `CAMetalLayer` — `nextDrawable()` → `CAMetalDrawable`

### Metal Shading Language (MSL)
- `vertex` / `fragment` / `kernel` qualifiers on entry point functions
- `[[attribute(n)]]` — vertex attribute binding index
- `[[buffer(n)]]` — buffer binding slot
- `[[texture(n)]]` / `[[sampler(n)]]` — resource binding slots
- `[[position]]` — vertex output position
- `constant` address space — uniform data
- `device` / `thread` address spaces
- `simd.h` types: `float2`, `float3`, `float4`, `float4x4`, etc.

## Code Highlights

Triple buffering with semaphore:
```swift
let inflightSemaphore = DispatchSemaphore(value: 3)
var bufferIndex = 0
let dynamicBuffers = (0..<3).map { _ in device.makeBuffer(length: bufferSize, options: .storageModeShared)! }

func draw(in view: MTKView) {
    inflightSemaphore.wait()
    bufferIndex = (bufferIndex + 1) % 3
    let buffer = dynamicBuffers[bufferIndex]
    // update buffer contents...
    let commandBuffer = commandQueue.makeCommandBuffer()!
    commandBuffer.addCompletedHandler { _ in
        self.inflightSemaphore.signal()
    }
    // encode render pass...
    commandBuffer.present(view.currentDrawable!)
    commandBuffer.commit()
}
```

MSL vertex shader (vs. GLSL):
```metal
struct VertexIn  { float3 position [[attribute(0)]]; float2 texCoord [[attribute(1)]]; };
struct VertexOut { float4 position [[position]]; float2 texCoord; };

vertex VertexOut vertexShader(VertexIn in [[stage_in]],
                               constant float4x4 &mvp [[buffer(1)]]) {
    VertexOut out;
    out.position = mvp * float4(in.position, 1.0);
    out.texCoord = in.texCoord;
    return out;
}
```

## Takeaways
- Metal's explicit command submission eliminates OpenGL's hidden validation stalls; pipeline state objects move expensive validation to initialization time.
- Triple-buffer dynamic resources with a `DispatchSemaphore`; avoid `waitUntilCompleted()` in the render loop.
- Compile all shaders at Xcode build time via `.metal` files — runtime shader compilation (like GLSL) is gone.
- Metal is now hardware-accelerated in the iOS/tvOS Simulator on macOS Catalina with Xcode 11, making development and debugging far more convenient.

---
_Source: WWDC19 Session 611 page (abstract, full transcript, and resource links including "Migrating OpenGL code to Metal" documentation)._
