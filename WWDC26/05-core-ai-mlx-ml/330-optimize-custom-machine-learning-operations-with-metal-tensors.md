# Optimize Custom Machine Learning Operations with Metal Tensors
**WWDC26 · Session 330** · [Watch](https://developer.apple.com/videos/play/wwdc2026/330/)

_Platforms:_ iOS, iPadOS, macOS, tvOS, visionOS

## Overview
This session covers how to write high-performance custom machine learning kernels using the Metal Tensor API and Metal Performance Primitives (MPP) TensorOps library, and how to integrate those kernels into Core AI applications. It is aimed at developers who need to go below Core AI and MLX to implement custom operations — novel attention variants, specialized quantized ops, or performance-critical kernels that aren't covered by higher-level primitives.

The session introduces quantized data types natively supported by TensorOps (including MX floating-point formats), the new multi-plane tensor abstraction for representing quantized data and scale factors together, and a complete implementation of Flash Attention using cooperative tensors, row reductions, and efficient memory reuse patterns. The final section shows how to plug a custom Metal TensorOps kernel into a Core AI model graph.

## Key Topics

### Apple's ML Software Stack
The stack from top to bottom: Core AI / MLX (framework level) → Metal Performance Shaders (MPS) → Metal Performance Primitives (MPP) → TensorOps → Metal shaders. Developers work at the TensorOps level when they need custom operations that MPS/MPP don't provide.

### Managing Quantized Data
Quantization reduces memory bandwidth requirements for large models — critical for fitting weights in GPU caches. TensorOps natively supports quantized data types including `MTLTensorDataTypeMetalFloat8E4M3` (MXFP8), `MTLTensorDataTypeMetalFloat8UE8M0` (scale format), and other quantized formats for M5 and A19 GPUs.

### Multi-Plane Tensors
A single `MTLTensor` object can now hold both quantized element data and scale factors as separate "planes." Configure via `MTLTensorAuxiliaryPlaneDescriptor` with `blockFactors` for the scale granularity, and `MTLTensorAuxiliaryPlaneDescriptorMap` to associate the scales plane (`MTLTensorPlaneTypeScales`) with the tensor descriptor.

### Quantized Matrix Multiplication
Demonstrates extending a tiled matrix multiplication kernel to support MXFP8 quantized inputs. In MSL: define type aliases for the multi-plane tensor handle and MSL `tensor<>` template specialization, use `tensor_inline` for stack-allocated tensors from buffer pointers, slice with threadgroup IDs, and call `matmul2d::run()` — TensorOps handles dequantization automatically.

### Building Advanced Ops: Flash Attention
Implements Flash Attention with TensorOps covering:
- SIMD group mappings via `execution_simdgroups<N>`
- **Cooperative tensors** — tiles that live in registers across a SIMD group, eliminating threadgroup memory round-trips
- `mul_qk_op.get_destination_cooperative_tensor<>()` — allocate QxK accumulator
- `reduce_rows(ctQK, ctTileRowMax, reduction_operation::max, -INFINITY)` — row-wise max for numerically stable SoftMax
- `map_iterator` — iterate parallel elements of two cooperative tensors for elementwise SoftMax
- `mul_sv_op.is_compatible_as_left_input<>(ctQK)` + `get_left_input_cooperative_tensor<>()` — reuse cooperative tensor directly as matmul input, bypassing threadgroup memory

### Integrating Custom Ops into Core AI
Custom Metal TensorOps kernels can be registered with Core AI's Python tooling during the conversion step, so they appear as named operations in the `.aimodel` graph. The Python converter specifies where custom Metal operations are used, and the Core AI runtime dispatches to the Metal kernel at inference time.

## APIs & Frameworks

**Metal Tensor API (Objective-C / MSL)** **[NEW for quantized types]**
- `MTLTensorDescriptor` — describes tensor shape, data type, usage
  - `.dataType` — e.g. `MTLTensorDataTypeMetalFloat8E4M3`, `MTLTensorDataTypeMetalFloat8UE8M0` **[NEW]**
  - `.usage` — `MTLTensorUsageCompute`
  - `.dimensions` — `MTLTensorExtents`
  - `.auxiliaryPlanes` — `MTLTensorAuxiliaryPlaneDescriptorMap` **[NEW]**
- `MTLTensorAuxiliaryPlaneDescriptor` — describes a scales or other auxiliary plane **[NEW]**
  - `.dataType` — data type for the auxiliary plane
  - `.blockFactors` — `MTLTensorExtents` specifying quantization block size
- `MTLTensorAuxiliaryPlaneDescriptorMap` — map of plane type to descriptor **[NEW]**
  - `setDescriptor(_:forPlane:)` — associate descriptor with `MTLTensorPlaneTypeScales` **[NEW]**
- `MTLTensorPlaneTypeScales` — plane type constant **[NEW]**
- `device.newTensorWithDescriptor(_:error:)` — create an `MTLTensor`

**Metal Shading Language (MSL) TensorOps**
- `#include <metal_tensor>` — TensorOps MSL header
- `tensor<DataType, Extents, Handle, ...AuxPlanes>` — multi-plane tensor handle type **[NEW]**
- `tensor_handle` — binding-based tensor handle (for kernel buffer bindings)
- `tensor_inline` — stack-allocated tensor from buffer pointers
- `tensor_blockwise<plane_type, data_type, block_size_n, block_size_m>` — scales plane descriptor
- `tensor_plane_scales` — tag for scales auxiliary plane
- `dextents<int, Rank>` — dynamic extents (rank-N)
- `tensor.slice(offset_n, offset_m)` — extract a tile from a tensor
- `matmul2d_descriptor(M, N, K, transposeLeft, transposeRight)` — compile-time matmul descriptor
- `matmul2d<descriptor, execution_simdgroups<N>>` — matmul operation type
  - `.run(tA, tB, tC)` — execute tiled matmul; auto-dequantizes quantized inputs **[NEW]**
  - `.get_destination_cooperative_tensor<>()` — allocate register-resident result tile **[NEW]**
  - `.get_row_reduction_destination_cooperative_tensor<>()` — row reduction output tile **[NEW]**
  - `.is_compatible_as_left_input<>()` — check if cooperative tensor can be reused **[NEW]**
  - `.get_left_input_cooperative_tensor<>()` — reuse cooperative tensor as matmul input **[NEW]**
- `reduce_rows(ct, ctOut, operation, identity)` — row-wise reduction over cooperative tensor **[NEW]**
  - `reduction_operation::max`, `reduction_operation::sum`
- Cooperative tensor iteration: `.begin()`, `.end()`, `.map_iterator(it)` **[NEW]**
- `execution_simdgroups<N>` — SIMD group execution policy
- `execution_simdgroup` — single SIMD group execution policy

**Core AI Python Integration**
- Custom Metal TensorOps ops registered at conversion time via Core AI Python tools
- Core AI runtime dispatches to Metal kernels at inference time

**Related Resources**
- [Running inline ML operations in a shader with Metal 4](https://developer.apple.com/documentation/Metal/running-inline-ml-operations-in-a-shader-with-metal-4)
- [Machine learning passes (Metal)](https://developer.apple.com/documentation/Metal/machine-learning-passes)
- [Metal Performance Primitives Programming Guide](https://developer.apple.com/download/files/Metal-Performance-Primitives-Programming-Guide.pdf)
- [Metal Performance Shaders](https://developer.apple.com/documentation/MetalPerformanceShaders)

## Code Highlights

Create a quantized multi-plane MTLTensor (Objective-C):
```objc
MTLTensorAuxiliaryPlaneDescriptor *planeDesc = [MTLTensorAuxiliaryPlaneDescriptor new];
planeDesc.dataType = MTLTensorDataTypeMetalFloat8UE8M0;
NSInteger blockFactors[2] = {32, 1};
planeDesc.blockFactors = [[MTLTensorExtents alloc] initWithRank:2 values:blockFactors];

MTLTensorAuxiliaryPlaneDescriptorMap *auxPlanes = [MTLTensorAuxiliaryPlaneDescriptorMap new];
[auxPlanes setDescriptor:planeDesc forPlane:MTLTensorPlaneTypeScales];

MTLTensorDescriptor *desc = [MTLTensorDescriptor new];
desc.dataType = MTLTensorDataTypeMetalFloat8E4M3;
desc.auxiliaryPlanes = auxPlanes;
```

Run quantized matmul in MSL (dequantization is automatic):
```metal
auto tA = matrixA.slice(0, tgid.y * TILEM);
auto tB = matrixB.slice(tgid.x * TILEN, 0);
auto tC = matrixC.slice(tgid.x * TILEN, tgid.y * TILEM);

constexpr auto descriptor = matmul2d_descriptor(TILEM, TILEN, dynamic_length_v<int>, false, false);
matmul2d<descriptor, execution_simdgroups<4>> op;
op.run(tA, tB, tC);  // handles MXFP8 dequantization automatically
```

Reuse cooperative tensor as Flash Attention matmul input:
```metal
if (mul_sv_op.is_compatible_as_left_input<float, half, float>(ctQK)) {
    auto ctQKIn = mul_sv_op.get_left_input_cooperative_tensor<float, half, float>(ctQK);
    mul_sv_op.run(ctQKIn, tVSlice, ctO);
}
```

## Takeaways
- Use TensorOps' multi-plane tensor API to work with quantized weights natively — it handles MXFP8 and MX scaling formats automatically in `matmul2d::run()`.
- Cooperative tensors eliminate the threadgroup memory round-trip in multi-stage operations like Flash Attention; always check `is_compatible_as_left_input` before falling back to threadgroup store/reload.
- Row reductions via `reduce_rows` paired with `map_iterator` provide an ergonomic pattern for numerically stable SoftMax without manual synchronization.
- Custom Metal TensorOps kernels integrate directly into Core AI model graphs — register them at conversion time so the Core AI runtime dispatches to your Metal code during inference.

---
_Source: WWDC26 Session 330 page (abstract, chapter summaries, code samples, and resource links)._
