# Maximize Your Metal Ray Tracing Performance
**WWDC22 · Session 10105** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10105/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13

## Overview
This session covers Metal 3 enhancements to the ray tracing API focused on performance and developer productivity. Three major themes are covered: new ray tracing features (per-primitive data, buffers from intersection function tables, ray tracing from indirect command buffers), acceleration structure improvements (up to 2.8x faster parallel builds on Apple silicon, new vertex formats, transformation matrices, heap allocation), and Xcode 14 Metal tooling enhancements (profiling of intersection/visible functions, shader debugging across linked functions, stack overflow detection in shader validation).

## Key Topics
- **Per-primitive data** — store small per-triangle/per-primitive data directly in the acceleration structure to eliminate complex auxiliary buffer lookups; accessed via `[[primitive_data]]` in Metal Shading Language; reduces memory indirections and cache misses; demonstrated 10–16% performance improvement in Apple's own test apps
- **Buffers from intersection function tables** — `intersectionFunctionTable.get_buffer<T>(index)` and `get_visible_function_table<T>(index)` allow the ray tracing kernel to access resources bound to the intersection function table, reducing duplicated bindings
- **Ray tracing from indirect command buffers** — enable via `MTLIndirectCommandBufferDescriptor.supportRayTracing = true`
- **Acceleration structure build performance** — up to 2.3x faster builds on Apple silicon; multiple small builds automatically parallelized (up to 2.8x faster when parallel); refitting up to 38% faster; all operations (build, compact, refit) benefit from parallelism
- **Parallel build guidelines** — use the same `MTLAccelerationStructureCommandEncoder` for all builds; rotate through a pool of scratch buffers (not one shared scratch buffer) to enable parallel execution
- **New vertex formats** — `MTLAccelerationStructureTriangleGeometryDescriptor.vertexFormat` now supports half-precision floats, 2-component formats for planar geometry, and normalized integer formats; previously only 3-component full-precision float was supported
- **Transformation matrices** — `MTLAccelerationStructureTriangleGeometryDescriptor.transformationMatrixBuffer` pre-transforms vertex data at build time; useful for normalizing compressed mesh data and for merging multiple objects into a single primitive acceleration structure to reduce instance count and improve intersection performance
- **Heap allocation of acceleration structures** — `MTLHeap.makeAccelerationStructure(descriptor:)` and `makeAccelerationStructure(size:)`; `MTLDevice.heapAccelerationStructureSizeAndAlign(descriptor:)`; replace many `useResource:` calls with a single `useHeap:` call for large scenes
- **Xcode 14 Metal tooling:**
  - Acceleration Structure Viewer: motion animation scrubber/playback, primitive highlight mode with per-primitive data inspector
  - Shader Profiler: now supports intersection functions, visible functions, and dynamic libraries
  - Shader Debugger: now supports linked visible functions and dynamic libraries
  - Shader Validation: new stack overflow detection for misconfigured `maxCallStackDepth` on pipeline descriptors

## APIs & Frameworks
**Metal — Ray Tracing (Metal 3 additions)**
- `MTLAccelerationStructureTriangleGeometryDescriptor.primitiveDataBuffer` **[NEW]** — `MTLBuffer` containing per-primitive data
- `MTLAccelerationStructureTriangleGeometryDescriptor.primitiveDataElementSize` **[NEW]** — size in bytes of each primitive's data
- `MTLAccelerationStructureTriangleGeometryDescriptor.primitiveDataStride` **[NEW]** — stride in bytes between primitive data entries
- `MTLAccelerationStructureTriangleGeometryDescriptor.primitiveDataBufferOffset` **[NEW]** — byte offset into the primitive data buffer
- `[[primitive_data]]` Metal Shading Language attribute **[NEW]** — intersection function argument receiving a pointer to the hit primitive's data; also available on `IntersectionResult.primitive_data`, `IntersectionQuery.get_candidate_primitive_data()`, `IntersectionQuery.get_committed_primitive_data()`
- `intersection_function_table.get_buffer<T>(index)` **[NEW]** — retrieves a buffer bound to an intersection function table
- `intersection_function_table.get_visible_function_table<FunctionType>(index)` **[NEW]** — retrieves a visible function table bound to an intersection function table
- `MTLIndirectCommandBufferDescriptor.supportRayTracing: Bool` **[NEW]** — enables ray tracing commands in an ICB
- `MTLAccelerationStructureTriangleGeometryDescriptor.vertexFormat: MTLAttributeFormat` **[NEW]** — vertex format for acceleration structure geometry (adds support for half, 2-component, and normalized integer formats)
- `MTLAccelerationStructureTriangleGeometryDescriptor.transformationMatrixBuffer: MTLBuffer?` **[NEW]** — pre-transform matrix buffer applied to vertices at build time
- `MTLAccelerationStructureTriangleGeometryDescriptor.transformationMatrixBufferOffset: Int` **[NEW]**
- `MTLPackedFloat4x3` — used to represent 4x3 transformation matrices
- `MTLHeap.makeAccelerationStructure(descriptor:) -> MTLAccelerationStructure?` **[NEW]** — allocates an acceleration structure from a heap using a descriptor
- `MTLHeap.makeAccelerationStructure(size:) -> MTLAccelerationStructure?` **[NEW]** — allocates an acceleration structure from a heap by size
- `MTLDevice.heapAccelerationStructureSizeAndAlign(descriptor:) -> MTLSizeAndAlign` **[NEW]** — queries the size and alignment needed for a heap-allocated acceleration structure
- `MTLCommandEncoder.useHeap(_:)` — single call replaces many `useResource:` calls for heap-allocated acceleration structures

**Metal — Existing (used in context)**
- `MTLAccelerationStructureCommandEncoder` — encodes build/refit/compact operations; use one encoder for all builds to enable parallel execution
- `MTLPrimitiveAccelerationStructureDescriptor`, `MTLInstanceAccelerationStructureDescriptor`
- `MTLIntersectionFunctionTable`

## Code Highlights
Per-primitive data struct and intersection function:
```metal
struct PrimitiveData {
    texture2d<float> texture;
    float2 uvs[3];
};

[[intersection(triangle, raytracing::triangle_data, raytracing::instancing)]]
bool alphaTestIntersection(float2 coordinates [[barycentric_coord]],
                     const device PrimitiveData *primitiveData [[primitive_data]]) {
    PrimitiveData ppd = *primitiveData;
    float2 UV = calculateSamplingCoords(coordinates, ppd.uvs[0], ppd.uvs[1], ppd.uvs[2]);
    float alpha = ppd.texture.sample(sam, UV).w;
    return alpha >= 0.5f;
}
```

Setting per-primitive data on geometry descriptor:
```swift
geometryDescriptor.primitiveDataBuffer = primitiveDataBuffer
geometryDescriptor.primitiveDataElementSize = MemoryLayout<PrimitiveData>.size
geometryDescriptor.primitiveDataStride = MemoryLayout<PrimitiveData>.stride
```

Parallel acceleration structure builds (use same encoder, pool of scratch buffers):
```swift
for (index, accelerationStructure) in accelerationStructures.enumerated() {
    encoder.build(accelerationStructure: accelerationStructure,
                  descriptor: descriptors[index],
                  scratchBuffer: scratchBuffers[index % numScratchBuffers],
                  scratchBufferOffset: 0)
}
```

Heap allocation:
```swift
let accelStruct = heap.makeAccelerationStructure(descriptor: descriptor)
// Or by size:
let sizeAndAlign = device.heapAccelerationStructureSizeAndAlign(descriptor: descriptor)
let accelStruct = heap.makeAccelerationStructure(size: sizeAndAlign.size)
```

## Takeaways
- Per-primitive data eliminates multi-level buffer indirection for intersection functions, improving both performance and code clarity; demonstrated 10–16% improvement in practice.
- Parallel acceleration structure builds on Apple silicon provide up to 2.8x speedup for many small builds — use one encoder and a rotation of scratch buffers to activate this.
- Heap-allocated acceleration structures enable replacing thousands of `useResource:` calls with a single `useHeap:` call, improving CPU performance for large scenes.
- Xcode 14's Metal tools now fully support profiling and debugging intersection functions, visible functions, and dynamic libraries — and shader validation catches stack overflow from misconfigured call stack depth.

---
_Source: WWDC22 Session 10105 page (abstract, chapter summaries, code samples, and resource links)._
