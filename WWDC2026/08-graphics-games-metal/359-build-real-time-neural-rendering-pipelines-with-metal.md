# Build Real-Time Neural Rendering Pipelines with Metal
**WWDC26 · Session 359** · [Watch](https://developer.apple.com/videos/play/wwdc2026/359/)

_Platforms:_ macOS, iOS, iPadOS, tvOS

## Overview
This session covers how to integrate machine learning directly into real-time rendering pipelines using Metal 4. Apple presents three progressive levels of adoption—from turnkey MetalFX neural denoising through custom-trained ML networks deployed via the Metal 4 command buffer, to fully inline tiny neural networks running inside Metal shaders using the new TensorOps API.

The session features a real-world case study from Maxon's Redshift Live, demonstrating how MetalFX Denoising enables path-traced rendering at one sample per pixel with production-quality results. This shows that ML-powered rendering is now practical for interactive and real-time workloads, not just offline pipelines.

The final section explores the most advanced integration path: training small specialized neural networks (such as sky visibility probes) that adapt to dynamic scenes each frame, executing both ML inference and online training within the same shader invocation that produces rendered output.

## Key Topics

### MetalFX Denoising (2:16)
Integrating MetalFX Denoising into a path tracer requires providing clean auxiliary inputs: albedo, depth, and motion vectors. Best practices include keeping these inputs noise-free (do not denoise the albedo buffer itself), handling transparency as a post-denoiser overlay, using the denoiser strength mask to control blending for stable regions, and applying primary surface replacement for mirror and glass surfaces to prevent the denoiser from blurring sharp reflections and refractions.

Motion vectors must account for the camera jitter used by temporal upscalers: subtract the jitter difference between the current and previous frame from the raw NDC delta to produce correct camera-only motion vectors for static geometry.

### Deploying Custom ML Networks with Metal 4 (9:57)
Developers can train neural networks offline (e.g., HDRNet for tone mapping) and export them to Metal Performance Shaders Graph format. These networks are then executed inside a Metal 4 command buffer using the ML command encoder, sitting directly alongside existing render and compute passes. This approach replaces complex post-processing chains (multiple bloom, exposure, color grading passes) with a single neural network inference call, reducing both code complexity and runtime cost.

### Inline Neural Networks with TensorOps (13:40)
The new TensorOps API enables building and evaluating multilayer perceptrons directly within Metal shaders using cooperative tensors. This supports both inference and online training within the same compute pass. The session demonstrates a sky visibility probe: a small MLP trained each frame to represent irradiance contributions from dynamic sky conditions, adapting in real time without a separate training pipeline. This eliminates round-trips through the CPU and enables the tightest possible integration between ML and rendering work.

## APIs & Frameworks

### Metal / Metal 4
- `MTLCommandBuffer` — command buffer for encoding render, compute, and ML passes together
- `MTLComputeCommandEncoder` — compute encoder used alongside ML encoder
- `MTLRenderCommandEncoder` — render encoder used alongside ML encoder
- **[NEW]** `MTL4::VisibilityOptionDevice` — visibility scope option for explicit barrier model
- **[NEW]** `barrierAfterStages(_:_:_:)` — Metal 4 explicit synchronization barrier
- `MTL::StageDispatch`, `MTL::StageAll` — pipeline stage tokens for barriers
- Metal Performance Primitives (MPP) — low-level building blocks for neural operations in shaders
- **[NEW]** TensorOps API — API for building and running small MLPs inline in Metal shaders
- Cooperative tensors — tensor types for use inside shader thread groups

### MetalFX
- `MTLFXTemporalScaler` / MetalFX Denoising — temporal denoiser for path-traced or noisy inputs
- Denoiser strength mask — per-pixel control over denoiser blend weight
- Primary surface replacement — technique to bypass denoiser for mirror/glass surfaces
- Auxiliary inputs: albedo buffer, depth buffer, motion vector buffer

### Metal Performance Shaders Graph (MPSGraph)
- `MPSGraph` — used to represent and execute trained neural networks
- ML command encoder — executes MPS Graph networks inline in a Metal 4 command buffer

### Metal Shading Language
- `metal_stdlib` — standard shader library
- NDC motion vector computation pattern (clip-space delta minus jitter delta)

## Code Highlights

Computing camera-only motion vectors with jitter correction (required for MetalFX Denoising):

```metal
float4 clipCurrent  = viewProjCurrent  * float4(worldPos, 1.0);
float4 clipPrevious = viewProjPrevious * float4(worldPos, 1.0);
float2 motion = (clipPrevious.xy / clipPrevious.w) - (clipCurrent.xy / clipCurrent.w);

float2 jitterCurrent  = getJitter(frameIndex);
float2 jitterPrevious = getJitter(frameIndexPrevious);
motion -= jitterPrevious - jitterCurrent;
```

## Takeaways
- MetalFX Denoising is the fastest on-ramp: clean auxiliary inputs and correct motion vectors are the key to production-quality results.
- Custom ML networks (e.g., neural tone mappers) can replace entire post-processing chains and execute inline in a Metal 4 command buffer with no CPU round-trip.
- The TensorOps / cooperative tensor API enables training and inference of tiny specialized networks directly inside compute shaders each frame.
- Start with MetalFX denoising for real-time titles, then advance to custom networks and inline TensorOps as rendering quality targets increase.

---
_Source: WWDC26 Session 359 page (abstract, chapter summaries, code samples, and resource links)._
