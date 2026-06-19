# Explore Hybrid Rendering with Metal Ray Tracing
**WWDC21 · Session 10150** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10150/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12, tvOS 15

## Overview
This session demonstrates how to combine rasterization with Metal ray tracing in a "hybrid rendering" architecture to produce significantly more accurate shadows, ambient occlusion, and reflections than screen-space techniques can achieve. The key insight is that ray tracing provides access to the full scene's acceleration structure from any shader stage, eliminating the screen-space limitations (missing off-screen occluders, depth-buffer aliasing, limited depth buffer coverage) of traditional rasterized approximations.

The session also introduces new Metal Debugger tools in Xcode 13: the Acceleration Structure viewer with multiple visualization modes (BVH traversal heat map, instance/geometry/intersection-function color coding), and support for capturing and inspecting ray-tracing work including intersection masks.

## Key Topics
- **Hybrid Rendering Architecture:** G-Buffer rasterization pass captures depth, normals, albedo. A subsequent ray-tracing compute pass reads the G-Buffer and queries the acceleration structure to produce shadows, AO, or reflection maps. A final accumulation pass composites everything.
- **Ray Tracing from Render Pipelines (NEW on Apple Silicon/iOS):** Dispatching ray-tracing work from a render pipeline lets all work execute in a single pass, keeping intermediate results on-chip tile memory and eliminating round-trips to system memory. Significantly reduces bandwidth and power consumption.
- **Ray-Traced Shadows:** Trace a shadow ray from a G-Buffer point toward the light direction. Use `accept_any_intersection(true)` on the intersector since any occluder is sufficient. Eliminates shadow map aliasing, shadow bias, per-light render passes, and off-frustum coverage issues. Supports transparent/translucent occluders via custom intersection functions.
- **Ray-Traced Ambient Occlusion:** Generate cosine-weighted hemisphere rays per pixel using G-Buffer normals. Set small `max_distance` (e.g., 0.5 m) to only check nearby neighborhood. Fraction of intersecting rays determines attenuation. Accurately handles geometry that is off-screen or nearly perpendicular to camera—major improvement over SSAO.
- **Ray-Traced Reflections:** Compute reflected incident vector over the G-Buffer normal, trace a ray in that direction. Shade the intersection point directly in the compute kernel using bindless scene data (requires Argument Buffer Tier 2 / bindless—see Session 10286). If no intersection, fall back to skybox sampling. Handles dynamic geometry with no cube map updates required.
- **Metal Debugger Acceleration Structure Viewer (NEW):** Visualize acceleration structures in 3D. Multiple modes: BVH traversal heat map, geometry/instance/intersection function color coding. Configurable intersector hints (culling, intersection masks) for interactive debugging of mask-based filtering bugs.

## APIs & Frameworks

**Metal**
- `MTLAccelerationStructure` – Scene acceleration structure bound to compute/render pipeline
- `intersector<triangle_data, instancing>` (MSL) – Ray intersector object
  - `accept_any_intersection(bool)` – Optimization for shadow rays (accept first hit)
  - `intersect(ray, acceleration_structure)` – Returns `intersection_result`
  - `intersection_result.type` – `.none`, `.triangle`, `.bounding_box`
  - `intersection_result.instance_id`, `.geometry_id`, `.primitive_id` – Navigation indices
- `ray` (MSL) – `ray(origin, direction, min_distance, max_distance)`
- Ray tracing from render pipeline stages **[NEW]** – Available on Apple Silicon and iOS devices; use tile shaders to dispatch ray tracing within a render pass to keep data on-chip
- `MTLRenderPassDescriptor.colorAttachments[n].storeAction = .store` – Required when G-Buffer must persist to next pass
- `MTLComputeCommandEncoder.setAccelerationStructure(_:bufferIndex:)` – Bind acceleration structure to compute kernel
- Intersection mask (`MTLAccelerationStructureInstanceDescriptor.mask`) – Per-instance bitmask; intersector tests with bitwise AND against ray mask

**Metal Debugger (Xcode 13)**
- Acceleration Structure Viewer **[NEW]** – 3D visualization of BVH and geometry
  - BVH traversal heat map visualization **[NEW]**
  - Instance / geometry / intersection-function color modes **[NEW]**
  - Intersector hint configuration (mask, culling) **[NEW]**
- Ray-tracing capture support **[NEW]** – Captures ray-tracing compute dispatches in Metal frame captures
- Labeled encoders and acceleration structures – Best practice for navigating captures

## Code Highlights
G-Buffer rasterization pass setup (Objective-C):
```objc
MTLRenderPassDescriptor *gbufferPass = [MTLRenderPassDescriptor new];
gbufferPass.depthAttachment.texture = gbuffer.depthTexture;
gbufferPass.depthAttachment.storeAction = MTLStoreActionStore;
gbufferPass.colorAttachments[0].texture = gbuffer.normalTexture;
gbufferPass.colorAttachments[0].storeAction = MTLStoreActionStore;

id<MTLRenderCommandEncoder> renderEncoder =
    [commandBuffer renderCommandEncoderWithDescriptor:gbufferPass];
encodeRenderScene(scene, renderEncoder);
[renderEncoder endEncoding];
```

Ray-tracing compute pass reading G-Buffer:
```objc
id<MTLComputeCommandEncoder> compEncoder = [commandBuffer computeCommandEncoder];
[compEncoder setTexture:gbuffer.depthTexture   atIndex:0];
[compEncoder setTexture:gbuffer.normalTexture  atIndex:1];
[compEncoder setTexture:outReflectionMap       atIndex:2];
[compEncoder setComputePipelineState:raytraceReflectionKernel];
encode2dDispatch(width, height, compEncoder);
[compEncoder endEncoding];
```

Shadow ray kernel (MSL):
```metal
float3 p = calculatePosition(depth_texture, thread_id);
ray shadowRay(p, lightDirection, 0.01f, 1.0f);

intersector<triangle_data, instancing> shadowIntersector;
shadowIntersector.accept_any_intersection(true);
auto intersection = shadowIntersector.intersect(shadowRay, accel_structure);

if (intersection.type == intersection_type::none) {
    // Point is illuminated by this light
}
```

Ambient occlusion kernel (MSL):
```metal
ray aoRay = cosineWeightedRay(thread_id); // random hemisphere ray along normal
aoRay.max_distance = 0.5f;
intersector<triangle_data, instancing> i;
auto intersection = i.intersect(aoRay, accel_structure);
if (intersection.type != intersection_type::none) {
    // Point is obscured — accumulate into attenuation factor
}
```

Reflection kernel (MSL):
```metal
float3 p = calculatePosition(depth_texture, thread_id); // world space
float3 reflectedDir = reflect(p - cameraPosition, normal);
ray reflectedRay(p, reflectedDir, 0.01f, FLT_MAX);
intersector<triangle_data, instancing> refIntersector;
auto intersection = refIntersector.intersect(reflectedRay, accel_structure);
if (intersection.type != intersection_type::none) {
    // Shade the reflected point (requires bindless scene data)
} else {
    // Sample skybox
}
```

## Takeaways
- Hybrid rendering replaces or supplements each light approximation pass with a ray-tracing query, producing more accurate results and in some cases eliminating the need for multiple extra render passes (e.g., per-light shadow maps).
- Ray tracing from render pipeline stages on Apple Silicon keeps all G-Buffer and ray-tracing intermediate data on-chip, avoiding system-memory bandwidth costs; this is a significant optimization worth targeting for iOS/macOS games.
- Use `accept_any_intersection(true)` for shadow rays—there is no need to find the closest intersection, just any occluder.
- Reflection kernels that need to shade intersection points require bindless access to scene geometry (Argument Buffer Tier 2); plan the scene data hierarchy before implementing reflection ray tracing.

---
_Source: WWDC21 Session 10150 page (abstract, transcript, and code samples)._
