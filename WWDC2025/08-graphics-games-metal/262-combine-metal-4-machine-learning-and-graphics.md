# Combine Metal 4 Machine Learning and Graphics
**WWDC25 · Session 262** · [Watch](https://developer.apple.com/videos/play/wwdc2025/262/)

_Platforms:_ iOS 26, iPadOS 26, macOS Tahoe 26

## Overview
Metal 4 introduces three integrated pillars for machine learning in graphics applications: **MTLTensor** (a new multi-dimensional GPU resource), **MTL4MachineLearningCommandEncoder** (for dispatching full ML networks on the GPU timeline alongside render and compute passes), and **Shader ML** with **Metal Performance Primitives** (for embedding small neural networks directly inside shaders). The session walks through the complete workflow — converting a PyTorch model to an MTLPackage, dispatching it inside a frame, implementing neural material compression in a fragment shader, and debugging ML workloads in the Metal Debugger with a new ML Network Debugger tool.

## Key Topics

### MTLTensor: The New ML Resource
`MTLTensor` is a first-class Metal resource alongside `MTLBuffer` and `MTLTexture`. It is a multi-dimensional container described by rank (number of axes) and extents (element count per axis), a `dataType`, and `usage` flags. Unlike `MTLBuffer`, `MTLTensor` bakes in stride and dimension metadata, simplifying indexing into high-rank data (e.g., convolution inputs). Usage flags: `MTLTensorUsageMachineLearning` for use with the ML encoder; `MTLTensorUsageCompute` or `MTLTensorUsageRender` for shader binding.

Tensors created from `MTLDevice` use an opaque, hardware-optimized layout (best performance). Tensors created from an existing `MTLBuffer` require explicit strides and support arbitrary padding.

### MTL4MachineLearningCommandEncoder
A new command encoder that brings entire ML networks onto the GPU timeline alongside render and compute encoders. Workflow:

**Offline:** Export a PyTorch model to CoreML (`.mlpackage`) using `coremltools`, then run `metal-package-builder` to produce an `.mtlpackage` — an optimized binary container.

**Runtime:**
1. Open the `.mtlpackage` as an `MTLLibrary`.
2. Describe the network via `MTL4MachineLearningPipelineDescriptor` (function name + input dimensions for dynamic shapes).
3. Compile a `MTL4MachineLearningPipelineState` once (expensive; cache it).
4. Create a `MTLHeap` with `type = MTLHeapTypePlacement` and `size ≥ pipeline.intermediatesHeapSize` for intermediate storage.
5. Encode with `machineLearningCommandEncoder()`, set the pipeline state, bind argument tables, call `dispatchNetworkWithIntermediatesHeap:`.

**Synchronization:** ML work participates in Metal 4 barrier and fence primitives. The new `MTLStageMachineLearning` stage value identifies ML workloads in barrier calls. Work that does not depend on ML output can run in parallel; only consuming passes need to wait.

### Shader ML and Metal Performance Primitives
For smaller networks that need to run per-fragment or per-thread, Shader ML lets you declare `tensor<>` values and call ML operations directly in Metal Shading Language 4.

Include `<metal_tensor>` to access the `tensor<>` type; include `<MetalPerformancePrimitives/MetalPerformancePrimitives.h>` for high-performance operations. Tensors are bound to shaders via buffer slots or argument buffers. Inline tensors (stack-allocated, from raw arrays) are also supported for small inputs with no strides needed.

**Metal Performance Primitives** provides:
- `tensor_ops::matmul2d<descriptor, execution_thread>` — matrix multiplication; parameterized by M/N/K sizes, transpose flags, and precision.
- Convolution operations.

The `execution_thread` template parameter means the operation runs on a single thread (required when per-fragment control flow is divergent). `execution_simdgroup` or `execution_threadgroup` can be used for uniform, non-divergent operations.

**Neural material compression use case:** Traditional fragment shaders sample albedo/normal textures. Neural compression instead samples latent textures, passes them through a two-layer MLP inside the fragment shader using `matmul2d`, and outputs decompressed material data — at 50% of block-compressed memory footprint with no perceptible quality loss.

### Debugging ML Workloads
The Metal Debugger in Xcode gains two key additions:
- **Dependency Viewer:** Shows all command buffers, encoders, barriers, events, and fences in a single graph view. Useful for validating that ML encoder synchronization is correct before investigating the network itself.
- **ML Network Debugger:** Opens an `MTL4MachineLearningCommandEncoder` dispatch as a visual DAG of operations. Individual operation outputs can be inspected as intermediate `MTLTensor` previews. Expands stitched composite operations to reveal constituent ops (e.g., an activation sequence). Enables bisecting a model to isolate which layer produces artifacts.

## APIs & Frameworks

**Metal 4 (iOS 26, macOS Tahoe 26)**
- **[NEW]** `MTLTensor` — multi-dimensional GPU resource for ML data
- **[NEW]** `MTLTensorDescriptor` — configures rank, extents, `dataType`, `usage`
- **[NEW]** `MTLTensorUsageMachineLearning`, `MTLTensorUsageCompute`, `MTLTensorUsageRender`
- **[NEW]** `MTLDevice.newTensorWithDescriptor:offset:error:` — allocate opaque tensor from device
- **[NEW]** `MTLBuffer.newTensorWithDescriptor:offset:error:` — wrap buffer data as tensor (requires strides)
- **[NEW]** `MTL4MachineLearningCommandEncoder` — ML network encoder on GPU timeline
- **[NEW]** `MTL4MachineLearningPipelineDescriptor` — describes function + dynamic input dimensions
- **[NEW]** `MTL4MachineLearningPipelineState` — compiled ML network; exposes `intermediatesHeapSize`
- **[NEW]** `MTLCommandBuffer.machineLearningCommandEncoder()` — creates ML encoder
- **[NEW]** `MTL4MachineLearningCommandEncoder.dispatchNetworkWithIntermediatesHeap:` — dispatch network
- **[NEW]** `MTLStageMachineLearning` — barrier stage constant for ML workloads
- **[NEW]** `metal-package-builder` — command-line tool; converts `.mlpackage` → `.mtlpackage`

**Metal Shading Language 4**
- **[NEW]** `<metal_tensor>` header — `tensor<>` type for shader ML
- **[NEW]** `tensor<device T, dextents<int, Rank>>` — dynamically-sized tensor bound via buffer slot
- **[NEW]** Inline tensor construction from arrays: `tensor(inputs, extents<int, N, 1>())`
- **[NEW]** `<MetalPerformancePrimitives/MetalPerformancePrimitives.h>` — Metal Performance Primitives library
- **[NEW]** `tensor_ops::matmul2d_descriptor` — configured with M, N, K, transpose flags, precision
- **[NEW]** `tensor_ops::matmul2d<descriptor, ExecutionGroup>` — matrix multiply in shader
- **[NEW]** `execution_thread`, `execution_simdgroup`, `execution_threadgroup` — execution group specifiers

**CoreML / Python tooling**
- `coremltools.convert(model, convert_to='mlprogram')` — export PyTorch/TensorFlow model to `.mlpackage`
- CoreML ML program format required (not older `.mlmodel`); minimum deployment target must be specified

**Xcode GPU Tools**
- **[NEW]** ML Network Debugger — visual DAG viewer for `MTL4MachineLearningCommandEncoder` dispatches; intermediate tensor inspection
- Dependency Viewer (enhanced) — now shows ML encoder, `MTLStageMachineLearning` barriers, and events

## Code Highlights
Create and dispatch an ML network on the GPU timeline (Objective-C):
```objc
// Offline: metal-package-builder mymodel.mlpackage -o myNetwork.mtlpackage

// Runtime
library = [device newLibraryWithURL:@"myNetwork.mtlpackage"];
MTL4LibraryFunctionDescriptor *funcDesc = [MTL4LibraryFunctionDescriptor new];
funcDesc.name = @"main";
funcDesc.library = library;

MTL4MachineLearningPipelineDescriptor *pipeDesc = [MTL4MachineLearningPipelineDescriptor new];
pipeDesc.machineLearningFunctionDescriptor = funcDesc;
[pipeDesc setInputDimensions:dimensions atBufferIndex:1];
pipeline = [compiler newMachineLearningPipelineStateWithDescriptor:pipeDesc error:&error];

// Intermediate heap
MTLHeapDescriptor *heapDesc = [MTLHeapDescriptor new];
heapDesc.type = MTLHeapTypePlacement;
heapDesc.size = pipeline.intermediatesHeapSize;
heap = [device newHeapWithDescriptor:heapDesc];

// Encode
encoder = [commands machineLearningCommandEncoder];
[encoder setPipelineState:pipeline];
[encoder setArgumentTable:argTable];
[encoder dispatchNetworkWithIntermediatesHeap:heap];
[commands endCommandBuffer];
[queue commit:&commands count:1];
```

Synchronize ML output with render pass:
```objc
[encoder barrierAfterStages:MTLStageMachineLearning
       beforeQueueStages:MTLStageVertex
       visibilityOptions:MTL4VisibilityOptionDevice];
```

Fragment shader with Shader ML (Metal Shading Language 4):
```metal
#include <metal_tensor>
#include <MetalPerformancePrimitives/MetalPerformancePrimitives.h>
using namespace mpp;

[[fragment]]
float4 shade_frag(tensor<device half, dextents<int, 2>> layer0Weights [[ buffer(0) ]],
                  tensor<device half, dextents<int, 2>> layer1Weights [[ buffer(1) ]]) {
    half inputs[INPUT_WIDTH] = { /* latent texture samples */ };
    auto inputTensor = tensor(inputs, extents<int, INPUT_WIDTH, 1>());

    constexpr tensor_ops::matmul2d_descriptor desc(1, HIDDEN_WIDTH, INPUT_WIDTH, false, true, true);
    tensor_ops::matmul2d<desc, execution_thread> op;
    op.run(inputTensor, layer0Weights, intermediate);

    // ReLU activation
    for (auto i = 0; i < HIDDEN_WIDTH; ++i)
        intermediate[i, 0] = max(0.0f, intermediate[i, 0]);

    // Second layer...
    half3 baseColor = half3(outputTensor[0,0], outputTensor[1,0], outputTensor[2,0]);
    return float4(baseColor * saturate(dot(worldNormal, lightDir)), 1.0);
}
```

## Takeaways
- Use `MTL4MachineLearningCommandEncoder` for full-frame ML workloads (upscaling, ambient occlusion, denoising) that need to synchronize precisely with render and compute work — Metal 4 barriers with `MTLStageMachineLearning` provide the necessary ordering without extra round-trips to the CPU.
- Use Shader ML and Metal Performance Primitives for per-fragment or per-thread inference (material decompression, neural shading) — combining sampling, inference, and shading into one dispatch eliminates device-memory synchronization between steps and cuts both memory bandwidth and disk footprint.
- Convert models to CoreML ML program format (`convert_to='mlprogram'`) before running `metal-package-builder`; older CoreML model files are not supported.
- Use the ML Network Debugger to bisect a model graph when output is incorrect — inspect intermediate `MTLTensor` values at any operation without modifying the model or adding debug instrumentation.

---
_Source: WWDC25 Session 262 page (abstract, chapter summaries, transcript, and code samples)._
