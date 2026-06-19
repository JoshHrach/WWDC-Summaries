# Discover Metal 3
**WWDC22 · Session 10066** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10066/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13 (Apple silicon, A13 Bionic+, M1+, recent AMD/Intel GPUs)

## Overview
Metal 3 is the next generation of Apple's low-overhead graphics and compute API, introducing a comprehensive set of new features that significantly raise performance and rendering quality for apps and games on Apple silicon. This overview session surveys all major Metal 3 capabilities: fast resource loading for high-quality asset streaming, offline shader compilation to eliminate load-time stutters, MetalFX Upscaling for high-resolution rendering at lower cost, mesh shaders for flexible geometry processing, enhanced ray tracing with GPU-driven pipelines, and accelerated machine learning for both inference and training.

Each feature area has a dedicated deep-dive session. This session is the entry point that explains the motivation, demonstrates the benefits, and directs developers to the right follow-on sessions. Metal 3 is supported on devices with the `.metal3` GPU family.

## Key Topics

### Fast Resource Loading
- Modern games need many small, low-latency asset loads, but existing storage APIs are optimized for large bulk requests.
- Metal 3 fast resource loading **[NEW]** provides explicit, multi-threaded loading commands directly into Metal buffers and textures without intermediate copies.
- Uses familiar Metal synchronization primitives (fences, semaphores) to coordinate with GPU work.
- Key benefit for sparse texture streaming: reduces time spent drawing with lower-quality textures by maximizing storage hardware throughput.
- See: "Load resources faster with Metal 3" (10104)

### Offline Shader Compilation
- Traditionally, shader binaries (GPU machine code) are generated at runtime during pipeline state object creation, causing loading delays and in-frame stutters.
- Metal 3 offline compilation **[NEW]** moves binary generation to project build time.
- Eliminates first-load stutter: all pipelines are fast to create at runtime because binaries are pre-compiled.
- Reduces app load times significantly when an app creates many complex pipelines.
- See: "Target and optimize GPU binaries with Metal 3" (10102)

### MetalFX Upscaling
- New framework **[NEW]**: MetalFX provides platform-optimized upscaling and anti-aliasing.
- Two modes: temporal (higher quality, uses motion data) and spatial (faster, single frame).
- Apps render at lower resolution and MetalFX generates high-quality full-resolution output at reduced GPU cost.
- Enables Retina-quality visuals with higher frame rates.
- See: "Boost performance with MetalFX Upscaling" (10103)

### Mesh Shaders
- New flexible 2-stage geometry pipeline replacing the traditional vertex stage **[NEW]**.
- **Object stage**: compute-like, evaluates whole objects, decides how many meshes to generate (culling, LOD selection).
- **Mesh stage**: generates actual geometry, sent directly to the rasterizer.
- No intermediate device memory needed for variable-size geometry output (unlike compute-preprocess + indirect draw approach).
- Ideal for: GPU-driven culling, LOD selection, procedural geometry generation.
- See: "Transform your geometry with Metal mesh shaders" (10162)

### Ray Tracing Improvements
- Acceleration structure build times reduced **[NEW]**.
- New Indirect Command Buffer (ICB) support for ray tracing enables GPU-driven ray tracing pipelines **[NEW]**.
- Direct access to primitive data for streamlined intersection and shading **[NEW]**.
- Reduces CPU overhead and enables more scalable ray tracing workloads.
- See: "Maximize your Metal ray tracing performance" (10105)

### Machine Learning Acceleration
- TensorFlow GPU acceleration on Mac: up to 16x speedup on M1 Ultra vs CPU; Metal 3 accelerates more TensorFlow ops with less CPU synchronization **[NEW]**.
- PyTorch GPU acceleration via Metal **[NEW]**: BERT training up to 6.5x faster, ResNet50 up to 8.5x faster on M1 Ultra vs CPU.
- ML inference acceleration for Metal-based video/image processing apps (e.g., DaVinci Resolve).
- See: "Accelerate machine learning with Metal" (10063)

### Developer Tools (Xcode 14)
- **Metal Dependency Viewer** — new synchronization edges to analyze GPU dependencies in GPU-driven and fast resource loading pipelines **[NEW]**.
- **Acceleration Structure Viewer** — highlight individual primitives, view primitive data, visualize motion information over time **[NEW]**.
- Other tools: Dylib support, new resource list, file navigation in Shader editor, custom Buffer Viewer layouts.

## APIs & Frameworks

### Metal 3 **[NEW]**
- `MTLDevice.supportsFamily(.metal3)` — capability check **[NEW]**
- `MTLIOCommandQueue` — fast resource loading command queue **[NEW]**
- `MTLIOCommandBuffer` — fast resource loading command buffer **[NEW]**
- `MTLIOFileHandle` — file handle for fast resource loading **[NEW]**
- Offline compilation pipeline: `metal` compiler with binary archiving at build time **[NEW]**
- `MTLBinaryArchive` — store and load precompiled shader binaries
- `MTLMeshRenderPipelineDescriptor` — mesh shader pipeline **[NEW]**
- Object stage / Mesh stage in render pipeline **[NEW]**
- Indirect Command Buffer (ICB) for ray tracing **[NEW]**
- Primitive data direct access in ray tracing intersection functions **[NEW]**
- Acceleration structure build optimizations **[NEW]**

### MetalFX **[NEW]**
- `MTLFXTemporalScaler` — temporal upscaling **[NEW]**
- `MTLFXSpatialScaler` — spatial upscaling **[NEW]**
- `MTLFXTemporalScalerDescriptor` — configure temporal upscaling
- `MTLFXSpatialScalerDescriptor` — configure spatial upscaling

### Metal Sparse Textures (existing, enhanced)
- `MTLSparseTexture` — tile-granularity texture streaming
- Coordinate with fast resource loading for tile loads

### ML Frameworks
- TensorFlow (with Metal GPU backend) — additional ops accelerated **[NEW]**
- PyTorch (with Metal GPU backend, MPS) — new GPU acceleration **[NEW]**
- Metal Performance Shaders (MPS) — inference acceleration

## Code Highlights

```swift
// Check Metal 3 support
if device.supportsFamily(.metal3) {
    // Use Metal 3 features
}

// MetalFX Spatial Upscaling setup
let desc = MTLFXSpatialScalerDescriptor()
desc.inputWidth = renderWidth
desc.inputHeight = renderHeight
desc.outputWidth = displayWidth
desc.outputHeight = displayHeight
desc.colorTextureFormat = .bgra8Unorm
let scaler = desc.makeSpatialScaler(device: device)!

// Mesh shader pipeline
let meshDesc = MTLMeshRenderPipelineDescriptor()
meshDesc.objectFunction = objectFunction
meshDesc.meshFunction = meshFunction
meshDesc.fragmentFunction = fragmentFunction
```

## Takeaways
- Metal 3 is a generational leap for Apple GPU programming, with fast resource loading, offline compilation, MetalFX, mesh shaders, improved ray tracing, and ML acceleration all shipping together.
- MetalFX Upscaling is the highest-impact single adoption for most games — significant frame rate improvements with minimal visual quality tradeoff.
- Offline compilation should be adopted by any app with a complex pipeline set to eliminate first-launch stutters.
- Check `device.supportsFamily(.metal3)` for capability gating; the feature set requires A13 Bionic or M1 or newer.

---
_Source: WWDC22 Session 10066 page (abstract, chapter summaries, code samples, and resource links)._
