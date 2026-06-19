# Go further with Metal 4 games

**Session ID:** 211  
**WWDC Year:** 2025  
**Folder:** `08-graphics-games-metal`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/211/

---

## Overview

This session is the advanced companion to "Explore Metal 4 games" (session 254), going deeper into Metal 4's capabilities for high-performance game rendering on Apple silicon. It covers multi-queue GPU scheduling, mesh shaders, hardware-accelerated ray tracing updates, and additional Metal 4 optimizations including argument buffers, shader function pointers, and the new offline compilation pipeline. It targets developers who have already adopted the baseline Metal 4 command encoding model and want to push performance further.

> Note: Full transcript data for this session was not available at summary time; details are derived from session metadata, chapter list, and Metal 4 framework documentation.

---

## Key Topics

- Multi-queue Metal 4 scheduling: graphics, compute, and async compute queues
- Mesh shaders: `MTLMeshRenderPipelineDescriptor` and the mesh/fragment shader stage
- Hardware ray tracing updates in Metal 4: intersection functions, motion blur
- Argument buffers tier 2 optimizations for bindless rendering
- Shader function pointers for dynamic dispatch in Metal shaders
- Offline compilation workflow: building `.metallib` binaries at build time with `metal` compiler toolchain
- Profiling Metal 4 workloads in Instruments: GPU Timeline, Shader Profiler updates
- MetalFX integration best practices for Metal 4 render loops

---

## APIs & Frameworks

- **Metal 4** (`Metal` framework) – **[NEW]** GPU API for Apple silicon; this session covers advanced features building on session 254.
- **`MTLCommandQueue` multi-queue** – **[NEW]** Metal 4 enables explicit multi-queue submission with independent graphics and async compute queues on the same `MTLDevice`; create via `device.makeCommandQueue()` with queue type descriptor.
- **`MTLMeshRenderPipelineDescriptor`** – **[NEW]** Pipeline descriptor for mesh shader pipelines; specifies an `objectFunction` (amplification), `meshFunction`, and `fragmentFunction`.
- **`MTLRenderCommandEncoder.drawMeshThreadgroups(_:threadsPerObjectThreadgroup:threadsPerMeshThreadgroup:)`** – **[NEW]** Draw call for mesh shader pipelines.
- **`MTLAccelerationStructure`** – Existing ray tracing type; Metal 4 updates: support for motion blur via `MTLMotionKeyframeData`, and instanced acceleration structures with per-instance transform updates at draw time.
- **`MTLIntersectionFunctionTable`** – Existing; updated in Metal 4 to support more complex intersection function signatures.
- **Argument buffers (Tier 2)** – Existing feature; in Metal 4, argument buffers can reference residency set resources directly for bindless rendering with no per-draw binding overhead.
- **Shader function pointers** – **[NEW]** Metal Shading Language feature allowing GPU-side dynamic dispatch; useful for material system implementations.
- **`MTLOfflineCompiler`** – **[NEW]** (see session 254); produces `.metallib` binaries; this session shows integrating the Metal compiler toolchain (`xcrun metal`) into Xcode build phases for automated offline compilation.
- **Metal Shader Profiler** (Instruments) – Updated in Xcode 26 to show per-function GPU time in mesh shader pipelines and async compute queues separately.
- **MetalFX `MTLFXTemporalScaler`** – Existing upscaling API; Metal 4 temporal scaler integration with multi-queue requires explicit `MTLFence` synchronization between the render queue and the MetalFX queue.
- **`MTLSharedEvent`** – Existing cross-queue/cross-process synchronization; used in Metal 4 multi-queue setups to coordinate graphics and compute submissions.

---

## Code Highlights

Creating a mesh shader pipeline:
```swift
import Metal

let desc = MTLMeshRenderPipelineDescriptor()
desc.objectFunction = library.makeFunction(name: "objectMain")
desc.meshFunction = library.makeFunction(name: "meshMain")
desc.fragmentFunction = library.makeFunction(name: "fragmentMain")
desc.colorAttachments[0].pixelFormat = .bgra8Unorm

let (pipeline, _) = try device.makeRenderPipelineState(descriptor: desc, options: [])
```

Dispatching a mesh shader draw:
```swift
renderEncoder.setRenderPipelineState(meshPipeline)
renderEncoder.drawMeshThreadgroups(
    MTLSize(width: objectCount, height: 1, depth: 1),
    threadsPerObjectThreadgroup: MTLSize(width: 1, height: 1, depth: 1),
    threadsPerMeshThreadgroup: MTLSize(width: 32, height: 1, depth: 1)
)
```

Multi-queue submission with synchronization:
```swift
let graphicsQueue = device.makeCommandQueue()!
let computeQueue = device.makeCommandQueue()!
let event = device.makeSharedEvent()!
var signalValue: UInt64 = 1

// Graphics queue signals event after rendering
let gfxBuffer = graphicsQueue.makeCommandBuffer()!
// ... encode render commands ...
gfxBuffer.encodeSignalEvent(event, value: signalValue)
gfxBuffer.commit()

// Compute queue waits for graphics to finish
let computeBuffer = computeQueue.makeCommandBuffer()!
computeBuffer.encodeWaitForEvent(event, value: signalValue)
// ... encode compute commands ...
computeBuffer.commit()
```

---

## Takeaways

- Mesh shaders replace the traditional vertex/geometry shader pipeline for geometry amplification workloads (foliage, LOD culling, particle systems); they run entirely on GPU with no CPU-side draw call amplification.
- Multi-queue scheduling allows the GPU's graphics and async compute engines to run simultaneously, filling otherwise idle shader cores during rasterization pipeline bubbles.
- Bindless rendering via Tier 2 argument buffers eliminates per-draw CPU binding overhead; combined with Metal 4 residency sets, this is the recommended architecture for large open-world scenes.
- Offline compilation is essential for shipping games: eliminate all first-play hitches by compiling every pipeline variant at build time and shipping `.metallib` binaries in the app bundle.
- Hardware ray tracing in Metal 4 now supports motion blur natively, removing the need for temporal approximations in games using ray-traced effects.
- Always profile with the Metal GPU Timeline and Shader Profiler in Instruments before and after adopting Metal 4 features; the gains are architecture-dependent.
