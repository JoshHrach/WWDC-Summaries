# Ray Tracing with Metal
**WWDC19 · Session 613** · [Watch](https://developer.apple.com/videos/play/wwdc2019/613/)

_Platforms:_ macOS Catalina 10.15, iOS 13

## Overview
Metal Performance Shaders (MPS) harnesses the GPU's massive parallelism to accelerate ray tracing and ray casting for both offline and real-time applications. This session covers the full ray tracing pipeline in Metal — from building and updating acceleration structures to denoising the final image — with emphasis on practical techniques for shipping real-time effects within tight performance budgets.

The session walks through three concrete rendering effects: hard shadows, soft shadows, and ambient occlusion, plus an introduction to global illumination. It also introduces two major new additions to the MPS ray tracing API: GPU-accelerated acceleration structure builds and a new spatiotemporal denoising system (MPSSVGF).

Multi-GPU support for ray tracing is also covered, enabling pro applications to distribute intersection work across multiple GPUs.

## Key Topics

### MPS Ray Tracing Pipeline Review
- Rays are defined by origin and direction, stored in Metal buffers.
- The `MPSRayIntersector` finds the closest triangle intersection along each ray and returns results (distance, triangle index, barycentric coordinates) in an intersection buffer.
- Acceleration structures partition scene geometry (triangles) spatially for fast traversal. Metal builds and manages these automatically.

### GPU-Accelerated Acceleration Structure Builds **[NEW]**
- Previously built on the CPU; now builds run on the GPU automatically when possible.
- Significant reduction in app startup cost with no code changes required.

### Dynamic Scenes
- **Vertex Animation / Refitting:** When geometry deforms (skinned meshes, cloth, etc.), Metal can refit the existing acceleration structure instead of rebuilding from scratch. Call `encodeRefit(into:)` — must enable refitting support before the initial build. Faster but degrades structure quality over time.
- **Rigid Body Animation — Two-Level Acceleration Structures:** Build per-unique-object triangle acceleration structures once; compose them via an `MPSInstanceAccelerationStructure`. Each instance has a transformation matrix and an index into the triangle acceleration structure array. Only the instance-level structure needs rebuilding per frame.

### Rendering Techniques
- **Hard Shadows:** Fire shadow rays from surface points toward the light source. Skip unnecessary rays by setting `maxDistance` to a negative value for background/back-facing pixels.
- **Soft Shadows:** Fire rays in a random cone toward the sun; use the hit/miss ratio to modulate shadow softness. Combined with the denoiser, one ray per pixel produces high-quality results.
- **Ambient Occlusion:** Fire rays in a hemisphere around the surface normal using cosine-weighted importance sampling. Bake a distance-squared falloff into the ray length distribution to concentrate short (cheap) rays where they matter most.
- **Global Illumination:** Full indirect lighting via path tracing, enabled by iterative ray bouncing with the same MPS infrastructure.

### Ray Buffer Layout Optimization
- Row-linear ray ordering causes cache thrashing in the acceleration structure traversal hardware.
- **Block-linear ordering** (8×8 pixel blocks) significantly improves GPU cache efficiency and throughput.
- Set `maxDistance < 0` to skip rays that don't need intersection testing.

### MPSSVGF Denoiser **[NEW]**
- Spatiotemporal denoiser that accumulates samples across multiple frames, guided by depth, normals, and motion vectors.
- Handles moving cameras and dynamic objects by invalidating stale history using depth/normal comparisons.
- Fast path for single-channel textures (ambient occlusion, shadow masks).
- Can denoise two independent image channels simultaneously.

### Multi-GPU Support **[NEW]**
- Ray intersection work can be distributed across multiple GPUs in Mac Pro configurations.

## APIs & Frameworks

### Metal Performance Shaders — Ray Tracing
- `MPSRayIntersector` — core ray/scene intersection accelerator
- `MPSRayOriginMinDistanceDirectionMaxDistance` — ray structure with origin, direction, min/max distance fields
- `MPSIntersectionDataTypeDistancePrimitiveIndexCoordinates` — intersection result type
- `MPSAccelerationStructure` — base acceleration structure type
- `MPSTriangleAccelerationStructure` — per-object triangle acceleration structure
- `MPSInstanceAccelerationStructure` **[NEW]** — two-level instanced acceleration structure
- `MPSAccelerationStructureGroup` **[NEW]** — groups structures that share resources in an instance hierarchy
- `encodeRefit(into:)` — GPU-side refitting for vertex-animated geometry **[NEW]**
- `allowRefitting` property — must be set before initial build to enable refitting

### Metal Performance Shaders — Denoising **[NEW]**
- `MPSSVGFDenoiser` — high-level spatiotemporal denoiser coordinator **[NEW]**
- `MPSSVGF` — low-level denoiser with individual compute kernels and tunable parameters **[NEW]**
- `MPSSVGFTextureAllocator` protocol — cache for temporary texture allocations during denoising **[NEW]**

### Metal (Core)
- `MTLCommandBuffer` — encodes acceleration structure builds, refits, ray intersection, and denoising
- `MTLBuffer` — holds ray structures and intersection results
- `MTLTexture` — depth, normal, motion vector, and output image textures
- Compute kernels (via `MTLComputeCommandEncoder`) — ray generation and shading passes

## Code Highlights

Enabling refitting before building:
```swift
accelerationStructure.allowRefitting = true
accelerationStructure.rebuild()
```

Refitting after vertex update:
```swift
intersector.encodeRefit(accelerationStructure, commandBuffer: commandBuffer)
```

Setting up a two-level instance acceleration structure:
```swift
let group = MPSAccelerationStructureGroup(device: device)
let instanceAS = MPSInstanceAccelerationStructure(group: group)
instanceAS.accelerationStructures = triangleAccelStructures
instanceAS.transformationMatrixBuffer = transformBuffer
instanceAS.instanceCount = numInstances
instanceAS.rebuild()
```

Skipping rays with negative maxDistance:
```metal
ray.maxDistance = isBackground ? -1.0 : INFINITY;
```

Setting up the denoiser:
```swift
let svgf = MPSSVGF(device: device)
let allocator = MPSSVGFDefaultTextureAllocator(device: device)
let denoiser = MPSSVGFDenoiser(MPSSVGF: svgf, textureAllocator: allocator)
denoiser.sourceTexture = noisyImage
denoiser.depthNormalTexture = depthNormal
denoiser.motionVectorTexture = motionVectors
denoiser.encode(to: commandBuffer)
let cleanImage = denoiser.destinationTexture
```

## Takeaways
- GPU-accelerated acceleration structure builds (automatic in iOS/macOS 13) eliminate a major CPU startup cost for MPS ray tracing apps.
- Use block-linear (8×8) ray buffer ordering and skip rays with `maxDistance < 0` for significant performance gains in real-time workloads.
- The new `MPSSVGFDenoiser` enables high-quality soft shadows and ambient occlusion with just one ray per pixel, making ray tracing viable in real-time apps.
- Two-level acceleration structures via `MPSInstanceAccelerationStructure` are the right tool for scenes with rigid-body animation or repeated geometry.

---
_Source: WWDC19 Session 613 page (abstract, chapter summaries, code samples, and resource links)._
