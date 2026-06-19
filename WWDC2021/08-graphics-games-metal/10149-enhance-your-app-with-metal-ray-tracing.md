# Enhance Your App with Metal Ray Tracing
**WWDC21 · Session 10149** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10149/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session covers three major areas of new Metal ray tracing capability in 2021: ray tracing from render pipeline stages (enabling hybrid rasterization + ray tracing in a single render pass), usability and portability improvements (intersection query, user instance IDs, instance transforms), and production rendering features (extended limits and physically accurate motion blur via keyframe animation).

The previous year's Metal ray tracing API required a separate compute pass for any ray tracing work. The new render pipeline integration eliminates that round-trip, allowing fragment shaders to trace rays directly against acceleration structures. The intersection query API provides an alternative to the intersector object for simpler cases, allowing in-line custom intersection logic without separate intersection functions. Production improvements include extended limits for large-scene rendering and native motion blur support using per-instance or per-primitive keyframe animation.

## Key Topics
- **Ray Tracing in Render Pipelines (NEW):** Fragment and vertex shaders can now directly intersect rays against acceleration structures without leaving the render pass. Intersection function tables are created per render stage. Especially powerful with tile functions on Apple Silicon (see Session 10150).
- **Intersection Query (NEW):** An alternative traversal API (`intersection_query`) that loops on `next()`, evaluates each candidate intersection inline, and commits passing candidates. Avoids the overhead of separate intersection functions and tables; trades some performance for simplicity. Ideal for porting from other ray tracing APIs.
- **User Instance IDs (NEW):** Custom 32-bit values attached to each instance via `MTLAccelerationStructureUserIDInstanceDescriptor`. Accessible in intersection functions via `[[user_instance_id]]` attribute and from intersection results and query committed results. Useful for per-instance material IDs, colors, flags.
- **Instance Transforms (NEW):** Object-to-world and world-to-object transformation matrices exposed from intersection functions (with `world_space_data` tag) and intersection results/queries.
- **Extended Limits (NEW):** `MTLAccelerationStructureUsageExtendedLimits` flag increases max primitives, geometries, instances, and mask bits. Set on acceleration structure descriptor and use `extended_limits` tag on intersector.
- **Motion Blur (NEW):** Physically accurate motion blur via keyframe animation at both instance level (`MTLAccelerationStructureInstanceDescriptorTypeMotion`, `motionTransformBuffer`) and primitive level (`MTLAccelerationStructureMotionTriangleGeometryDescriptor`, per-keyframe `vertexBuffers`). Ray time is sampled randomly within the exposure interval and passed to the intersector.

## APIs & Frameworks

**Metal**
- Ray tracing in render pipeline stages **[NEW]**
  - `fragmentLinkedFunctions` / `vertexLinkedFunctions` on `MTLRenderPipelineDescriptor` – Attach intersection functions to render stages
  - `newIntersectionFunctionTableWithDescriptor:stage:` – Creates table for specific render stage **[NEW]**
  - `functionHandleWithFunction:stage:` – Gets handle for specific render stage **[NEW]**
  - `setFragmentAccelerationStructure:atBufferIndex:` on `MTLRenderCommandEncoder` **[NEW]**
  - `setFragmentIntersectionFunctionTable:atBufferIndex:` on `MTLRenderCommandEncoder` **[NEW]**
- `MTLAccelerationStructureUserIDInstanceDescriptor` **[NEW]** – Instance descriptor with custom `userID` field
  - `accelDesc.instanceDescriptorType = .userID`
- `MTLAccelerationStructureUsageExtendedLimits` **[NEW]** – Extended scene size limits
- `MTLAccelerationStructureInstanceDescriptorTypeMotion` **[NEW]** – Motion instance descriptor type
- `MTLInstanceAccelerationStructureDescriptor.motionTransformBuffer` **[NEW]**
- `MTLInstanceAccelerationStructureDescriptor.motionTransformCount` **[NEW]**
- `MTLAccelerationStructureMotionTriangleGeometryDescriptor` **[NEW]** – Per-keyframe vertex buffers
- `MTLMotionKeyframeData` **[NEW]** – Wraps a buffer + offset for a single keyframe
- `MTLPrimitiveAccelerationStructureDescriptor.motionKeyframeCount` **[NEW]**

**Metal Shading Language**
- `intersection_query<tags>` **[NEW]** – Inline traversal object replacing intersector for simple cases
  - `iq.next()` – Advances traversal, returns `false` when done
  - `iq.get_candidate_intersection_type()` – `intersection_type::triangle` or `::bounding_box`
  - `iq.get_candidate_geometry_id()`, `iq.get_candidate_primitive_id()`, `iq.get_candidate_triangle_barycentric_coord()`
  - `iq.commit_triangle_intersection()` – Commits a triangle candidate
  - `iq.get_committed_intersection_type()` – `.none`, `.triangle`, `.bounding_box`
  - `iq.get_committed_instance_id()`, `iq.get_committed_distance()`, `iq.get_committed_triangle_barycentric_coord()`
  - `iq.get_committed_user_instance_id()` **[NEW]**
  - `iq.get_committed_object_to_world_transform()` / `iq.get_committed_world_to_object_transform()` **[NEW]**
- `[[user_instance_id]]` attribute in intersection functions **[NEW]**
- `[[object_to_world_transform]]` / `[[world_to_object_transform]]` attributes in intersection functions **[NEW]**
- `intersector<extended_limits>` **[NEW]** – Extended limits tag
- `intersector<instance_motion>` / `acceleration_structure<instance_motion>` **[NEW]** – Instance motion tag
- `intersector<primitive_motion>` / `acceleration_structure<primitive_motion>` **[NEW]** – Primitive motion tag
- `intersector.intersect(ray, as, time)` **[NEW]** – Intersect with time parameter for motion blur
- `intersection_result<instancing>.user_instance_id` **[NEW]**
- `intersection_result<instancing, world_space_data>.object_to_world_transform` **[NEW]**

## Code Highlights
Creating intersection function table for a render pipeline fragment stage:
```objc
MTLIntersectionFunctionTableDescriptor *tableDesc = [MTLIntersectionFunctionTableDescriptor new];
tableDesc.functionCount = functions.count;
id<MTLIntersectionFunctionTable> table =
    [pipeline newIntersectionFunctionTableWithDescriptor:tableDesc
                                                  stage:MTLRenderStageFragment];
```

Inline alpha test using intersection query (MSL):
```metal
intersection_query<instancing, triangle_data> iq(ray, as, params);
while (iq.next()) {
    if (iq.get_candidate_intersection_type() == intersection_type::triangle) {
        if (alphaTest(iq.get_candidate_geometry_id(), ...))
            iq.commit_triangle_intersection();
    }
}
```

Sampling time for motion blur (MSL):
```metal
float time = random(exposure_start, exposure_end);
result = intersector.intersect(ray, acceleration_structure, time);
```

## Takeaways
- Ray tracing in render pipeline stages eliminates the compute-pass round-trip for hybrid rendering, enabling more efficient algorithms like tile-memory-based deferred ray tracing on Apple Silicon.
- `intersection_query` is ideal for simple custom intersection logic (alpha test, etc.) and for porting from Vulkan/DXR; for complex intersection use cases the `intersector` object remains better.
- User instance IDs let you attach arbitrary 32-bit metadata per instance directly in the acceleration structure, reducing the need for external lookup tables during shading.
- Native motion blur support uses keyframe animation at both instance and primitive levels, enabling physically accurate blur in offline rendering as well as interactive applications.

---
_Source: WWDC21 Session 10149 page (abstract, chapter summaries, code samples, and resource links)._
