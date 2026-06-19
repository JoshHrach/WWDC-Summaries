# Discover Ray Tracing with Metal
**WWDC20 · Session 10012** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10012/)

_Platforms:_ macOS Big Sur 11, iOS 14, iPadOS 14 (GPU family dependent; see Metal Feature Set Tables)

## Overview
iOS 14 / macOS Big Sur introduce a native Metal ray tracing API that exposes a ray intersector directly from within any compute kernel written in the Metal Shading Language (MSL). Previously, ray tracing on Apple platforms required the Metal Performance Shaders (MPS) framework with a separate `MPSRayIntersector` class, which forced developers to split their rendering logic into three distinct compute kernels and shuttle rays and intersection results through GPU memory buffers. The new API collapses this into a single compute kernel.

The session covers the complete ray tracing pipeline: the `intersector` object in MSL, building and using primitive and instance acceleration structures from Swift/Objective-C, and writing intersection functions — both triangle intersection functions (for effects like alpha testing) and bounding box intersection functions (for custom primitives like spheres, curves, and hair). A demo on a Mac Pro renders a 32 million–triangle scene assembled from Quixel Megascans assets using path tracing.

## Key Topics

### The Metal Ray Tracing API vs. MPS
- **MPS approach**: three separate compute kernels (generate rays → intersect with `MPSRayIntersector` → shade); rays and intersections passed through Metal buffers.
- **New Metal approach**: single compute kernel with an `intersector` object available inline in MSL; no intermediate buffers needed; outer bounce loop expressed as a simple `for`/`while` loop in the shader.
- Trade-off: combining intersection with complex shading may reduce occupancy; profile to find the best kernel division for your workload.

### Acceleration Structures **[NEW]**
Two types:
- **`MTLPrimitiveAccelerationStructure`** — contains primitives (triangles or bounding boxes); built from geometry descriptors.
- **`MTLInstanceAccelerationStructure`** — two-level instancing of primitive acceleration structures; reduces memory usage for repeated geometry.

Build pipeline:
1. Create a descriptor (`MTLPrimitiveAccelerationStructureDescriptor`).
2. Attach geometry descriptors (`MTLAccelerationStructureTriangleGeometryDescriptor` or `MTLAccelerationStructureBoundingBoxGeometryDescriptor`).
3. Query sizes with `device.accelerationStructureSizes(descriptor:)`.
4. Allocate `MTLAccelerationStructure` and scratch `MTLBuffer` (`.storageModePrivate`).
5. Encode build command via `MTLAccelerationStructureCommandEncoder` on any command queue — runs entirely on GPU timeline with no CPU synchronization.

Additional capabilities: **refitting** (for dynamic geometry), **compaction** (reclaim memory after build), **instancing**.

### MSL Ray Intersector
- `intersector<triangle_data>` — template type; `.intersect(ray, accelerationStructure)` returns `intersection_result<triangle_data>`.
- `intersector.assume_geometry_type(.triangles)` — hint for shadow/AO rays that accept first intersection.
- `primitive_acceleration_structure` and `instance_acceleration_structure` — MSL buffer types for acceleration structures.
- `intersection_function_table<triangle_data>` — MSL buffer type for intersection function tables.
- Bind via `computeEncoder.setAccelerationStructure(_:bufferIndex:)` and `computeEncoder.setIntersectionFunctionTable(_:bufferIndex:)`.

### Intersection Functions **[NEW]**
Two flavors:
1. **Triangle intersection functions** (`[[intersection(triangle, triangle_data)]]`) — invoked per triangle hit; return `bool` to accept/reject; receive `primitive_id`, `geometry_id`, `barycentric_coord`; bind own resources. Use for alpha testing, decals, etc.
2. **Bounding box intersection functions** (`[[intersection(bounding_box)]]`) — invoked per bounding box hit; must compute and return the intersection distance; model arbitrary custom primitives (spheres, NURBS, Bézier curve hair strands). Return a struct with `[[accept_intersection]]` bool and `[[distance]]` float.

**Ray payload**: a `ray_data`-qualified reference parameter shared between the intersection function and the calling compute kernel; allows passing data (e.g., surface normals) back without additional buffers.

**Linking**: intersection functions are linked into a compute pipeline state via `MTLLinkedFunctions`; function handles are retrieved from the pipeline state and inserted into `MTLIntersectionFunctionTable` entries that map to geometry via `intersectionFunctionTableOffset`.

## APIs & Frameworks

### Metal (macOS/iOS) **[NEW in 2020]**
- `MTLDevice.accelerationStructureSizes(descriptor:)` → `MTLAccelerationStructureSizes` (`.accelerationStructureSize`, `.buildScratchBufferSize`, `.refitScratchBufferSize`)
- `MTLDevice.makeAccelerationStructure(size:)` → `MTLAccelerationStructure`
- `MTLAccelerationStructureCommandEncoder` — `build(accelerationStructure:descriptor:scratchBuffer:scratchBufferOffset:)`, `refit(...)`, `copy(...)`, `compact(...)`
- `MTLPrimitiveAccelerationStructureDescriptor` — `.geometryDescriptors: [MTLAccelerationStructureGeometryDescriptor]`
- `MTLAccelerationStructureTriangleGeometryDescriptor` — `.vertexBuffer`, `.indexBuffer`, `.triangleCount`, `.intersectionFunctionTableOffset`
- `MTLAccelerationStructureBoundingBoxGeometryDescriptor` — `.boundingBoxBuffer`, `.boundingBoxCount`, `.intersectionFunctionTableOffset`
- `MTLInstanceAccelerationStructureDescriptor` — `.instanceDescriptorBuffer`, `.instanceCount`
- `MTLAccelerationStructureInstanceDescriptor` — struct with `transformationMatrix`, `mask`, `intersectionFunctionTableOffset`, `accelerationStructureIndex`
- `MTLLinkedFunctions` — `.functions: [MTLFunction]`
- `MTLComputePipelineDescriptor.linkedFunctions`
- `MTLComputePipelineState.makeIntersectionFunctionTable(descriptor:)` → `MTLIntersectionFunctionTable`
- `MTLComputePipelineState.functionHandle(function:)` → `MTLFunctionHandle`
- `MTLIntersectionFunctionTableDescriptor` — `.functionCount`
- `MTLIntersectionFunctionTable.setFunction(_:index:)`, `.setBuffer(_:offset:index:)`, `.setTexture(_:index:)`
- `MTLComputeCommandEncoder.setAccelerationStructure(_:bufferIndex:)` **[NEW]**
- `MTLComputeCommandEncoder.setIntersectionFunctionTable(_:bufferIndex:)` **[NEW]**

### Metal Shading Language (MSL) **[NEW types/attributes]**
- `intersector<geometry_type, ...>` — inline intersection object
- `intersection_result<geometry_type>` — intersection query result (`.distance`, `.primitive_id`, `.geometry_id`, `.triangle_barycentric_coord`, `.accepted`)
- `primitive_acceleration_structure` — buffer binding type
- `instance_acceleration_structure` — buffer binding type
- `intersection_function_table<geometry_type>` — buffer binding type
- `[[intersection(triangle, triangle_data)]]` — triangle intersection function attribute
- `[[intersection(bounding_box)]]` — bounding box intersection function attribute
- `[[primitive_id]]`, `[[geometry_id]]`, `[[barycentric_coord]]` — triangle intersection inputs
- `[[origin]]`, `[[direction]]`, `[[min_distance]]`, `[[max_distance]]` — ray data inputs
- `[[accept_intersection]]`, `[[distance]]` — bounding box result struct attributes
- `ray_data` — address space qualifier for ray payload parameters

## Code Highlights

Basic ray tracing compute kernel:
```metal
[[kernel]]
void rtKernel(primitive_acceleration_structure accelStruct [[buffer(0)]], ...) {
    ray r = generateCameraRay(tid);
    intersector<triangle_data> i;
    intersection_result<triangle_data> result = i.intersect(r, accelStruct);
    // shade using result...
}
```

Triangle alpha-test intersection function:
```metal
[[intersection(triangle, triangle_data)]]
bool alphaTestFn(uint primitiveIndex [[primitive_id]],
                 uint geometryIndex  [[geometry_id]],
                 float2 bary         [[barycentric_coord]],
                 device Material *materials [[buffer(0)]]) {
    float alpha = materials[geometryIndex].alphaTexture.sample(s, interpolateUVs(...)).x;
    return alpha > 0.5f;
}
```

Building a primitive acceleration structure:
```swift
let desc = MTLPrimitiveAccelerationStructureDescriptor()
let geomDesc = MTLAccelerationStructureTriangleGeometryDescriptor()
geomDesc.vertexBuffer = vertexBuffer; geomDesc.triangleCount = count
desc.geometryDescriptors = [geomDesc]
let sizes = device.accelerationStructureSizes(descriptor: desc)
let accelStruct = device.makeAccelerationStructure(size: sizes.accelerationStructureSize)!
let scratch = device.makeBuffer(length: sizes.buildScratchBufferSize, options: .storageModePrivate)!
let enc = commandBuffer.makeAccelerationStructureCommandEncoder()!
enc.build(accelerationStructure: accelStruct, descriptor: desc, scratchBuffer: scratch, scratchBufferOffset: 0)
enc.endEncoding(); commandBuffer.commit()
```

## Takeaways
- The new Metal ray tracing API replaces multi-kernel MPS ray tracing with a single compute kernel containing an inline `intersector` — eliminating ray/intersection buffer round-trips and enabling simpler, more flexible shaders.
- Acceleration structures are now fully developer-controlled allocations built on the GPU timeline with no CPU sync required; refitting and compaction are supported.
- Triangle intersection functions enable efficient alpha testing and similar per-hit customization without restarting traversal; bounding box intersection functions enable fully custom primitive types (spheres, curves, hair).
- Ray payloads allow intersection functions to pass arbitrary data (e.g., surface normals) back to the calling compute kernel.

---
_Source: WWDC20 Session 10012 page (abstract, transcript, code samples, and resource links)._
