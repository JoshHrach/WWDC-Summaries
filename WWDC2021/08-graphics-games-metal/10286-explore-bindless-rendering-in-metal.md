# Explore Bindless Rendering in Metal
**WWDC21 · Session 10286** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10286/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
Bindless rendering is a modern GPU resource binding model that makes an entire scene's resources—textures, vertex buffers, index buffers, material constants—available to shaders at once through a single bound root buffer, rather than binding each resource individually before each draw call. This session explains why bindless is necessary for certain ray tracing effects (reflections, diffuse GI), how to construct a bindless scene hierarchy using Argument Buffers Tier 2, how to declare residency for indirectly accessed resources, and how to navigate the scene structure from Metal Shading Language shaders to retrieve vertex attributes and shade intersection points correctly.

The session also covers how the same bindless architecture applies to rasterization pipelines, specifically PBR fragment shading, where bindless removes per-draw-call slot limits and enables efficient GPU-driven rendering with instanced draw calls.

## Key Topics
- **Why Bindless for Ray Tracing:** Shadow rays only need world-space position, but reflection/GI rays need vertex data and material data for any object in the acceleration structure. Since a ray can hit any object, it is impossible to bind all potentially needed resources through the traditional slot model. Bindless solves this by encoding all scene resources into linked argument buffers.
- **Argument Buffers Tier 2 (Requirement):** Bindless requires Tier 2, available on Apple6 and Mac2 GPU families. Argument buffers can be used from all Metal shader types (kernel, vertex, fragment), making them usable for both ray tracing and rasterization.
- **Scene Hierarchy Design:** Encode scene as linked argument buffers: `Instances` array (each with a `Mesh*`, `Material*`, and inline `float4x4` transform), `Mesh` (with `indices` array and per-attribute vertex arrays), `Material` (with texture references and inline constants). Bind the root `Scene*` pointer to the pipeline; all other resources are accessed indirectly.
- **Residency Declaration:** Indirectly accessed resources must be made resident explicitly. Use `useResource:usage:` (compute) or `useResource:usage:stages:` (render). Resources on `MTLHeap` can be made resident in a single `useHeap:` call. Read-only static resources (textures, meshes) should be placed on a heap at load time for minimal per-frame overhead.
- **Heap and Hazard Tracking:** Heaps can be tracked (Metal 2.3+) or untracked (default). Tracked heaps synchronize at heap granularity, causing false sharing between unrelated suballocations. Recommended approach: separate heaps for read-only static data (use `useHeap`) and writable/dynamic resources (use per-resource `useResource` with write flag). Use stage-granularity fences for maximum GPU parallelism.
- **Encoder Creation via MTLArgumentDescriptor:** When the argument buffer is indirectly referenced (not a direct function parameter), or when encoding arrays, create the encoder via `MTLArgumentDescriptor` array passed to `[MTLDevice newArgumentEncoderWithArguments:]`. Encoder offset advances by `encodedLength` per array element.
- **Navigation in MSL:** Use `intersection.instance_id`, `intersection.geometry_id`, and `intersection.primitive_id` from the intersection result object to navigate the bindless hierarchy. Pull three vertex indices, retrieve per-vertex attributes, and interpolate via barycentric coordinates.
- **Bindless for Rasterization (PBR):** Bind the root scene buffer once; navigate to the correct `Material` in the fragment shader by instance. Removes per-draw texture slot rebinding and enables instanced rendering to reduce draw call count.

## APIs & Frameworks

**Metal**
- `MTLArgumentDescriptor` – Describes an argument buffer struct member for encoder creation **[NEW usage pattern]**
  - `MTLArgumentDescriptor.argumentDescriptor` – Factory method
  - `.index` – Binding index matching MSL `[[id(N)]]` attribute
  - `.dataType` – `MTLDataTypePointer`, `MTLDataTypeTexture`, `MTLDataTypeFloat4x4`, etc.
  - `.access` – `MTLArgumentAccessReadOnly`, `MTLArgumentAccessReadWrite`
- `MTLDevice.newArgumentEncoderWithArguments([MTLArgumentDescriptor])` – Creates encoder without requiring MTLFunction **[NEW usage pattern]**
- `id<MTLArgumentEncoder>.encodedLength` – Stride of one encoded element; used for array encoding
- `id<MTLArgumentEncoder>.setArgumentBuffer(_:offset:)` – Sets the target buffer and offset
- `MTLComputeCommandEncoder.useResource(_:usage:)` – Declares residency for indirectly accessed resource
- `MTLRenderCommandEncoder.useResource(_:usage:stages:)` – Declares residency with stage granularity
- `MTLComputeCommandEncoder.useHeap(_:)` – Makes all heap suballocations resident in one call
- `MTLRenderCommandEncoder.useHeap(_:stages:)` – Heap residency with stage granularity
- `MTLHeap` – Suballocation heap; supports tracked (hazard-safe) or untracked (manual sync) mode
- Argument Buffers Tier 2 – Required for bindless; available on Apple6 / Mac2 GPU families
- `MTLAccelerationStructure` – Contains `instance_id`, `geometry_id`, `primitive_id` accessible from intersection results

**Metal Shading Language**
- `intersection_result<triangle_data, instancing>` – Intersection result type providing `.instance_id`, `.geometry_id`, `.primitive_id`, `.type`
- `intersection_type::triangle` – Checks for valid triangle intersection
- `device const Scene*` – Root bindless scene pointer bound to buffer slot
- `[[id(N)]]` attribute – Binds struct member to argument buffer slot N
- Barycentric interpolation – Manual `weightedSum(n0, n1, n2, barycentrics)` over fetched vertex attributes

**Companion Sample Code**
- `Rendering reflections in real time using ray tracing` – Apple sample demonstrating full bindless scene encoding with Model I/O, ray-traced reflections, hybrid rasterization + ray tracing

## Code Highlights
Bindless PBR fragment shader (traditional vs. bindless):
```metal
// Traditional: bind each texture to a slot before every draw call
fragment half4 pbrFragment(ColorInOut in [[stage_in]],
                           texture2d<float> albedo    [[texture(0)]],
                           texture2d<float> roughness [[texture(1)]],
                           texture2d<float> metallic  [[texture(2)]],
                           texture2d<float> occlusion [[texture(3)]]) {
    return calculateShading(in, albedo, roughness, metallic, occlusion);
}

// Bindless: navigate to material via scene pointer
fragment half4 pbrFragmentBindless(ColorInOut in [[stage_in]],
                                   device const Scene* pScene [[buffer(0)]]) {
    device const Instance& instance = pScene->instances[in.instance_id];
    device const Material& material = pScene->materials[instance.material_id];
    return calculateShading(in, material);
}
```

Argument buffer encoder creation via MTLArgumentDescriptor (Objective-C):
```objc
MTLArgumentDescriptor *meshArg = [MTLArgumentDescriptor argumentDescriptor];
meshArg.index    = 0;
meshArg.dataType = MTLDataTypePointer;
meshArg.access   = MTLArgumentAccessReadOnly;
// Declare materialArg (index 1) and transformArg (index 2) similarly
id<MTLArgumentEncoder> instanceEncoder =
    [device newArgumentEncoderWithArguments:@[meshArg, materialArg, transformArg]];
```

Ray tracing navigation (MSL):
```metal
constant Instance& instance = pScene->instances[intersection.instance_id];
constant Mesh&     mesh     = instance.mesh[intersection.geometry_id];
ushort3 indices;
indices.x = mesh.indices[intersection.primitive_id * 3 + 0];
indices.y = mesh.indices[intersection.primitive_id * 3 + 1];
indices.z = mesh.indices[intersection.primitive_id * 3 + 2];
packed_float3 n0 = mesh.normals[indices.x];
packed_float3 n1 = mesh.normals[indices.y];
packed_float3 n2 = mesh.normals[indices.z];
float3 barycentrics = calculateBarycentrics(intersection);
float3 normal = weightedSum(n0, n1, n2, barycentrics);
```

## Takeaways
- Bindless rendering is mandatory (not optional) for visually correct reflections and other effects that require attribute access at arbitrary ray-hit locations in the acceleration structure.
- Design the bindless scene hierarchy to mirror the acceleration structure topology (Instance → Geometry/Mesh → Primitive) so that `instance_id`, `geometry_id`, and `primitive_id` map directly to navigation steps.
- Always call `useResource:` or `useHeap:` for all indirectly accessed resources before encoding draw/dispatch calls; accessing a non-resident resource causes GPU restarts and command buffer failures.
- The same bindless scene buffers work for both ray tracing kernels and rasterization fragment shaders, enabling a unified scene representation that avoids duplication.

---
_Source: WWDC21 Session 10286 page (abstract, transcript, and code samples)._
