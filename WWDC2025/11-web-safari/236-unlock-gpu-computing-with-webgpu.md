# Unlock GPU Computing with WebGPU
**WWDC25 · Session 236** · [Watch](https://developer.apple.com/videos/play/wwdc2025/236/)

_Platforms:_ Safari 26 (iOS 26, iPadOS 26, macOS Tahoe 26)

## Overview
WebGPU is the modern web standard for GPU-accelerated graphics and compute, available in Safari 26. This session introduces the WebGPU API, explains its relationship to Metal, and walks through building GPU pipelines for both rendering and compute. Topics include the WGSL shading language, device and resource creation, render and compute pipelines, render bundles for performance optimization, bind groups for resource binding, and the `shader-f16` extension for half-precision float performance.

## Key Topics

### What is WebGPU?
WebGPU is a W3C web API that exposes GPU capabilities to JavaScript. Unlike WebGL (which maps to OpenGL ES), WebGPU maps closely to modern low-level GPU APIs: Metal on Apple platforms, Vulkan on Linux/Android, and D3D12 on Windows. It supports both graphics (vertex + fragment shaders) and general-purpose compute (compute shaders).

### GPUDevice
`GPUDevice` is the root object for all WebGPU operations. Obtained by calling `adapter.requestDevice()`, it is used to create buffers, textures, shaders, pipelines, and command encoders. Most WebGPU resources are created through the device.

### WGSL (WebGPU Shading Language)
WGSL is the shading language for WebGPU, replacing GLSL. It is statically typed with explicit address spaces (`storage`, `uniform`, `workgroup`, `private`, `function`), structured control flow, and shader stage annotations (`@vertex`, `@fragment`, `@compute`). Shaders are passed as strings to `device.createShaderModule()`.

### Render Pipeline
A `GPURenderPipeline` is created from a descriptor specifying vertex and fragment shader modules, vertex buffer layouts, primitive topology, and output color/depth formats. Drawing is encoded into a `GPURenderPassEncoder` obtained from a `GPUCommandEncoder`.

### Compute Pipeline
A `GPUComputePipeline` uses a single compute shader with `@compute @workgroup_size(x, y, z)`. Compute passes encode `dispatchWorkgroups(x, y, z)` calls. Compute enables general GPGPU work: image processing, physics simulation, ML inference.

### Render Bundles
`GPURenderBundle` records a reusable sequence of render commands. Replaying a bundle is significantly faster than re-encoding commands each frame, particularly useful for static scene geometry.

### Bind Groups
`GPUBindGroup` represents a set of resources (buffers, textures, samplers) bound together and passed to a shader. Created from a `GPUBindGroupLayout` (declared in the pipeline descriptor), bind groups are the primary mechanism for communicating CPU data to shaders.

### shader-f16 Extension
The `shader-f16` WebGPU extension enables half-precision (`f16`) floating-point types in WGSL. On Apple Silicon, f16 arithmetic is natively supported by the GPU and provides a significant performance boost for ML inference and image processing workloads. Enable by requesting the `"shader-f16"` feature in `requestDevice()` and using `enable f16;` at the top of WGSL shaders.

## APIs & Frameworks

**WebGPU (W3C Web API, Safari 26)**
- `GPUAdapter` — represents the physical GPU; obtained via `navigator.gpu.requestAdapter()`
- `GPUDevice` — root resource-creation object; obtained via `adapter.requestDevice(features: ["shader-f16"])`
- `GPUBuffer` — GPU-side memory buffer; created via `device.createBuffer()`
- `GPUTexture` / `GPUTextureView` — GPU textures for rendering and sampling
- `GPUSampler` — texture sampling parameters
- `GPUShaderModule` — compiled WGSL shader; created via `device.createShaderModule({ code: wgslSource })`
- `GPURenderPipeline` **[NEW in Safari 26]** — graphics pipeline (vertex + fragment); created via `device.createRenderPipeline()`
- `GPUComputePipeline` **[NEW in Safari 26]** — compute pipeline; created via `device.createComputePipeline()`
- `GPUBindGroupLayout` / `GPUBindGroup` — resource binding abstraction; created via `device.createBindGroupLayout()` / `device.createBindGroup()`
- `GPUCommandEncoder` — records GPU commands; obtained via `device.createCommandEncoder()`
- `GPURenderPassEncoder` — encodes render commands within a pass
- `GPUComputePassEncoder` — encodes compute dispatch commands
- `GPURenderBundle` / `GPURenderBundleEncoder` **[NEW]** — reusable pre-recorded render commands for performance
- `shader-f16` WebGPU feature **[NEW in Safari 26]** — half-precision float support in WGSL (`f16`, `vec2h`, `mat4x4h`, etc.)
- `GPUCanvasContext` — integrates WebGPU output with HTML `<canvas>`

**WGSL (WebGPU Shading Language)**
- `@vertex`, `@fragment`, `@compute` shader stage annotations
- `@workgroup_size(x, y, z)` — compute workgroup dimensions
- Address spaces: `storage`, `uniform`, `workgroup`, `private`, `function`
- `enable f16;` directive for half-precision float support

## Code Highlights

```javascript
// Initialize WebGPU with f16 support
const adapter = await navigator.gpu.requestAdapter();
const device = await adapter.requestDevice({ requiredFeatures: ["shader-f16"] });

// Create a shader module
const shaderModule = device.createShaderModule({ code: `
  enable f16;
  @vertex fn vs(@builtin(vertex_index) i: u32) -> @builtin(position) vec4f { ... }
  @fragment fn fs() -> @location(0) vec4f { return vec4f(1, 0, 0, 1); }
`});

// Create render pipeline
const pipeline = device.createRenderPipeline({
  layout: "auto",
  vertex: { module: shaderModule, entryPoint: "vs" },
  fragment: { module: shaderModule, entryPoint: "fs", targets: [{ format: "bgra8unorm" }] }
});
```

```javascript
// Compute dispatch
const computePipeline = device.createComputePipeline({
  layout: "auto",
  compute: { module: shaderModule, entryPoint: "main" }
});
const encoder = device.createCommandEncoder();
const pass = encoder.beginComputePass();
pass.setPipeline(computePipeline);
pass.setBindGroup(0, bindGroup);
pass.dispatchWorkgroups(64, 64);
pass.end();
device.queue.submit([encoder.finish()]);
```

## Takeaways
- WebGPU in Safari 26 provides access to the full GPU pipeline (graphics + compute) from JavaScript with performance close to native Metal applications.
- Use compute shaders for GPGPU tasks (image processing, ML inference, physics) that previously required WebAssembly or CPU-based workarounds.
- Adopt `GPURenderBundle` for scene geometry that doesn't change every frame — it dramatically reduces command-encoding overhead.
- Request the `shader-f16` feature for f16-heavy workloads (ML inference, signal processing) to take advantage of Apple Silicon's native half-precision GPU arithmetic.
- WGSL is the only shading language for WebGPU; GLSL is not supported. Write new shaders in WGSL and migrate any WebGL shader code as part of a WebGPU adoption.

---
_Source: WWDC25 Session 236 page (abstract, chapter summaries, code samples, and resource links)._
