# Transform Your Geometry with Metal Mesh Shaders
**WWDC22 · Session 10162** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10162/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13 (Family7 and Mac2 GPU families)

## Overview
Metal mesh shaders introduce a new programmable geometry pipeline that replaces the vertex shader stage with two new stages: an **object shader** and a **mesh shader**. This pipeline enables GPU-driven geometry creation and processing — removing the need for separate compute passes, intermediate buffers, and CPU-side draw call encoding — and is available on Apple Family7 (A15+) and Mac2 (M1+) GPUs.

Two key use cases are presented: (1) procedural geometry generation such as hair/fur rendering, where each GPU thread group autonomously generates a full strand of hair as a `metal::mesh` in a single render pass; and (2) GPU-driven meshlet culling, where the object shader performs frustum culling and only dispatches mesh shaders for visible meshlets, collapsing what was previously a compute pass + barrier + render pass into a single render command encoder.

## Key Topics

### Traditional Pipeline vs. Mesh Shaders
The traditional vertex → rasterizer → fragment pipeline requires separate compute command encoders for GPU-driven geometry, plus device-memory buffers to hold generated geometry. Mesh shaders eliminate the cross-encoder barrier and intermediate buffer by moving geometry generation into the render pass itself.

### Object Shader Stage
Launched as a grid of thread groups, each processing one "object" (abstract unit defined by the developer — could be a scene model, a tile, or a region of space). Each object thread group:
- Generates a payload (up to 16 KB) passed to the mesh grid
- Programmatically sets the mesh grid size via `mesh_grid_properties.set_threadgroups_per_grid()`
- Maximum objects per dispatch: 1024 mesh thread groups per object thread group

### Mesh Shader Stage
Each thread group in the mesh grid receives the payload and produces a `metal::mesh` — vertex data, index data, primitive data, and primitive count — all written directly to the rasterizer. No device memory allocation required.

### metal::mesh Type
A built-in MSL structure parameterized by vertex type, primitive type, max vertex count (≤256), max primitive count (≤512), and topology (point, line, triangle). Total size must not exceed 16 KB. Methods: `set_vertex`, `set_index`, `set_primitive`, `set_primitive_count`.

### Hair Rendering Use Case
A plane is divided into tiles; each tile becomes an object thread group that generates curve control points as payload and launches a mesh grid. Each mesh thread group generates one strand of hair as a triangle mesh. No additional buffers or compute passes needed.

### GPU-Driven Meshlet Culling Use Case
The object shader performs frustum culling and LOD selection for meshlets (small patches of triangles), dispatching mesh shaders only for visible meshlets. Eliminates the traditional compute pass → barrier → render pass pattern, improving GPU occupancy and reducing memory bandwidth.

### Pipeline Setup
`MTLMeshRenderPipelineDescriptor` replaces `MTLRenderPipelineDescriptor`. Draw calls use `drawMeshThreadgroups(_:threadsPerObjectThreadgroup:threadsPerMeshThreadgroup:)`.

## APIs & Frameworks

**Metal** (all **[NEW]** on Family7 / Mac2 in iOS 16 / macOS 13)

_Metal Shading Language (MSL)_
- `[[object]]` attribute **[NEW]** — marks a function as an object shader
- `[[mesh]]` attribute **[NEW]** — marks a function as a mesh shader
- `object_data` address space **[NEW]** — used for payload arguments
- `[[payload]]` attribute **[NEW]** — marks the payload output parameter in object shaders
- `mesh_grid_properties` **[NEW]** — struct with `set_threadgroups_per_grid(_:)` method
- `metal::mesh<VertexType, PrimitiveType, maxVertices, maxPrimitives, Topology>` **[NEW]** — output type for mesh shaders
- `metal::topology::triangle` / `::line` / `::point` **[NEW]** — topology options for `metal::mesh`
- `metal::mesh.set_vertex(_:_:)` **[NEW]**
- `metal::mesh.set_index(_:_:)` **[NEW]**
- `metal::mesh.set_primitive(_:_:)` **[NEW]**
- `metal::mesh.set_primitive_count(_:)` **[NEW]**

_Metal API (Swift/Obj-C)_
- `MTLMeshRenderPipelineDescriptor` **[NEW]** — pipeline descriptor for mesh pipelines
  - `.objectFunction: MTLFunction?`
  - `.meshFunction: MTLFunction?`
  - `.fragmentFunction: MTLFunction?`
  - `.payloadMemoryLength: Int`
  - `.maxTotalThreadsPerObjectThreadgroup: Int`
  - `.maxTotalThreadsPerMeshThreadgroup: Int`
- `MTLDevice.makeRenderPipelineState(descriptor: MTLMeshRenderPipelineDescriptor, options:)` **[NEW]**
- `MTLRenderCommandEncoder.drawMeshThreadgroups(_:threadsPerObjectThreadgroup:threadsPerMeshThreadgroup:)` **[NEW]**
- `MTLRenderCommandEncoder.setObjectBuffer(_:offset:index:)` **[NEW]**
- `MTLRenderCommandEncoder.setObjectTexture(_:index:)` **[NEW]**
- `MTLRenderCommandEncoder.setObjectSamplerState(_:index:)` **[NEW]**
- `MTLRenderCommandEncoder.setMeshBuffer(_:offset:index:)` **[NEW]**
- `MTLRenderCommandEncoder.setMeshTexture(_:index:)` **[NEW]**

## Code Highlights

MSL object shader for hair generation:
```metal
[[object]]
void objectShader(object_data CurvePayload *payloadOutput [[payload]],
                  const device void *inputData [[buffer(0)]],
                  uint hairID [[thread_index_in_threadgroup]],
                  uint triangleID [[threadgroup_position_in_grid]],
                  mesh_grid_properties mgp)
{
    if (hairID < kHairsPerBlock)
        payloadOutput[hairID] = generateCurveData(inputData, hairID, triangleID);
    if (hairID == 0)
        mgp.set_threadgroups_per_grid(uint3(kHairPerBlockX, kHairPerBlockY, 1));
}
```

MSL mesh type declaration and mesh shader:
```metal
using triangle_mesh_t = metal::mesh<VertexData, PrimitiveData, 10, 6, metal::topology::triangle>;

[[mesh]] void myMeshShader(triangle_mesh_t outputMesh,
                            uint tid [[thread_index_in_threadgroup]])
{
    if (tid < kVertexCount)   outputMesh.set_vertex(tid, calculateVertex(tid));
    if (tid < kIndexCount)    outputMesh.set_index(tid, calculateIndex(tid));
    if (tid < kPrimitiveCount) outputMesh.set_primitive(tid, calculatePrimitive(tid));
    if (tid == 0)             outputMesh.set_primitive_count(kPrimitiveCount);
}
```

Swift pipeline setup and draw call:
```swift
let meshPipelineDescriptor = MTLMeshRenderPipelineDescriptor()
meshPipelineDescriptor.objectFunction = objectFunction
meshPipelineDescriptor.meshFunction = meshFunction
meshPipelineDescriptor.fragmentFunction = fragmentFunction
meshPipelineDescriptor.payloadMemoryLength = payloadLength
meshPipelineDescriptor.maxTotalThreadsPerObjectThreadgroup = hairsPerBlock

let (meshPipeline, _) = try device.makeRenderPipelineState(descriptor: meshPipelineDescriptor, options: [])

encoder.setRenderPipelineState(meshPipeline)
encoder.setObjectBuffer(objectBuffer, offset: 0, index: 0)
encoder.drawMeshThreadgroups(
    MTLSize(width: trianglesPerModel, height: 1, depth: 1),
    threadsPerObjectThreadgroup: MTLSize(width: hairsPerBlock, height: 1, depth: 1),
    threadsPerMeshThreadgroup: MTLSize(width: threadsPerHair, height: 1, depth: 1))
```

## Takeaways
- Metal mesh shaders replace the vertex shader with a two-stage programmable pipeline (object + mesh), enabling fully GPU-driven geometry creation in a single render pass with no intermediate buffers.
- `metal::mesh` is the fundamental output type: each mesh shader thread group fills vertex, index, and primitive data directly for the rasterizer, up to 256 vertices and 512 primitives per mesh.
- Mesh shaders are available only on Family7 (Apple A15 and later) and Mac2 (M1 and later) GPU families; check device capability before using.
- Both use cases — procedural geometry (hair/fur) and meshlet culling — benefit from eliminating the compute-pass/barrier/render-pass chain, reducing memory allocation and improving GPU pipeline utilization.

---
_Source: WWDC22 Session 10162 page (abstract, chapter summaries, code samples, and resource links)._
