# Your guide to Metal ray tracing
**WWDC23 · Session 10128** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10128/)

_Platforms:_ iOS 17, iPadOS 17, macOS Sonoma 14, visionOS 1 (Metal — Apple GPU), macOS (AMD/Intel, compute only)

## Overview
This session is a comprehensive guide to the Metal ray tracing API covering both games and production rendering. It walks through the full pipeline: building primitive acceleration structures from triangle, bounding box, and the new curve geometry descriptors; scaling scenes with instance acceleration structures including new multi-level instancing (up to N levels deep) and a new GPU-driven indirect instance descriptor; optimizing builds with parallelization, refitting, and compaction; and intersecting rays with the Metal Shading Language `intersector` API including new curve intersection and multi-level instance results. The session also covers Shader Validation improvements, the Acceleration Structure viewer in Xcode, Shader Debugger, and the profiling timeline.

## Key Topics

**Acceleration Structure Fundamentals**
- Three geometry descriptor types: triangle, bounding box (custom intersection function), and **curves** (new)
- Curve primitives ideal for hair, fur, vegetation — smaller memory footprint and faster builds than triangle approximations
- Build pipeline: create descriptor → query sizes → allocate from heap → build with scratch buffer on GPU via `MTLAccelerationStructureCommandEncoder`

**Curve Geometry (NEW)**
- Supported curve basis functions: Bezier (2–4 control points), Catmull-Rom, B-Spline, Linear
- Cross-section types: round (3D cylindrical) and flat (better performance for distant geometry)
- Control point index buffer maps one index per segment to the first control point in that segment; segments on the same curve share control points
- Memory savings vs. triangles come from shared control points and compact representation
- `MTLAccelerationStructureCurveGeometryDescriptor` **[NEW]**: `controlPointBuffer`, `radiusBuffer`, `indexBuffer`, `controlPointCount`, `segmentCount`, `curveType` (`.round`/`.flat`), `curveBasis` (`.bezier`, `.catmullRom`, `.bSpline`, `.linear`), `segmentControlPointCount`

**Instance Acceleration Structures**
- Two-level model: instance AS → references primitive ASes with per-instance transformation matrices
- `MTLInstanceAccelerationStructureDescriptor`: `instanceCount`, `instancedAccelerationStructures`, `instanceDescriptorType`, `instanceDescriptorBuffer`
- Instance descriptor types: `.default`, `.userID`, `.motion`, `.indirect`
- Indirect instance descriptor **[NEW]**: `MTLIndirectInstanceAccelerationStructureDescriptor` with `maxInstanceCount` and `instanceCountBuffer` (GPU writes final count) — enables GPU-driven culling without CPU round-trip

**Multi-Level Instancing (NEW)**
- Instance AS can now reference other instance ASes (not just primitive ASes)
- Enables hierarchical scene composition: tree = trunk primitive + leaf instances; forest = tree instances
- Disney Moana Island Scene example: 3 levels, saving millions of instances vs. two-level approach
- Also valuable for games: separate static and dynamic ASes; 3-level instancing reduces rebuild time with only minor trace-time cost
- `max_levels<N>` tag in Metal Shading Language `intersector` type specifies depth

**Build Optimization**
- **Parallelization**: encode multiple builds to the same `MTLAccelerationStructureCommandEncoder`; scratch buffers are reusable after each build completes
- **Refitting**: `commandEncoder.refit(sourceAccelerationStructure:descriptor:destinationAccelerationStructure:scratchBuffer:scratchBufferOffset:)` — updates hierarchy for minor geometry changes without full rebuild; cheaper than rebuild
- **Compaction**: `writeCompactedSize(accelerationStructure:buffer:offset:sizeDataType:)` + `copyAndCompact(sourceAccelerationStructure:destinationAccelerationStructure:)` — reduces memory footprint after build; especially valuable for primitive ASes

**Ray Intersection (Metal Shading Language)**
- Bind AS: `encoder.setAccelerationStructure(_:bufferIndex:)` (primitive or instance)
- For instance AS: also call `encoder.useHeap(_:)` or `useResource(_:usage:)` to make referenced ASes resident
- `intersector<tags>` and `intersection_result<tags>` template types; tags: `triangle_data`, `instancing`, `max_levels<N>`, `curve_data`
- `intersector.intersect(ray, accelerationStructure)` → `intersection_result`
- `intersection_result.type` — `.triangle`, `.boundingBox`, `.curve`
- Curve-specific: `intersection_result.curve_parameter` (plug into basis function); `intersector.assume_geometry_type()`, `.assume_curve_type()`, `.assume_curve_basis()`, `.assume_curve_control_point_count()` — hint for better performance when all curves are the same type

**Debugging and Profiling**
- Shader Validation now covers all Metal ray tracing APIs **[NEW coverage]**; significantly reduced impact on shader compilation time
- Acceleration Structure viewer in Xcode: outline view of AS hierarchy, viewport with highlight modes (AABB traversal depth, AS coloring, primitive coloring); supports multi-level instancing and curve geometry
- Shader Debugger: per-variable value inspection, neighboring thread values shown in data view
- Profiling timeline: per-pipeline-state cost, per-line shader profiling with cost breakdown pie charts

## APIs & Frameworks

**Metal (Objective-C / Swift)**
- `MTLAccelerationStructureCurveGeometryDescriptor` **[NEW]**
- `MTLCurveType` **[NEW]**: `.round`, `.flat`
- `MTLCurveBasis` **[NEW]**: `.bezier`, `.catmullRom`, `.bSpline`, `.linear`
- `MTLIndirectInstanceAccelerationStructureDescriptor` **[NEW]**
- `MTLIndirectInstanceAccelerationStructureDescriptor.maxInstanceCount` **[NEW]**
- `MTLIndirectInstanceAccelerationStructureDescriptor.instanceCountBuffer` **[NEW]**
- `MTLAccelerationStructureInstanceDescriptorType.indirect` **[NEW]**
- `MTLAccelerationStructureCommandEncoder.refit(sourceAccelerationStructure:descriptor:destinationAccelerationStructure:scratchBuffer:scratchBufferOffset:)` (existing, noted)
- `MTLAccelerationStructureCommandEncoder.writeCompactedSize(accelerationStructure:buffer:offset:sizeDataType:)` (existing)
- `MTLAccelerationStructureCommandEncoder.copyAndCompact(sourceAccelerationStructure:destinationAccelerationStructure:)` (existing)
- `MTLAccelerationStructureSizes.refitScratchBufferSize` (existing)
- `MTLDevice.accelerationStructureSizes(descriptor:)` (existing)
- `MTLDevice.heapAccelerationStructureSizeAndAlign(size:)` (existing)

**Metal Shading Language**
- `acceleration_structure<instancing, max_levels<N>>` **[NEW tag]** — multi-level instance type
- `intersector<instancing, max_levels<N>, curve_data>` **[NEW tags]**
- `intersection_result<instancing, max_levels<N>, curve_data>` **[NEW tags]**
- `intersection_result.instance_count` **[NEW]** — number of intersected instances in multi-level result
- `intersection_result.instance_id[i]` **[NEW]** — ID of each intersected instance
- `intersection_result.curve_parameter` **[NEW]** — parameter for basis function evaluation
- `intersector.assume_geometry_type(geometry_type::curve | ...)` **[NEW]**
- `intersector.assume_curve_type(curve_type::round)` **[NEW]**
- `intersector.assume_curve_basis(curve_basis::bezier)` **[NEW]**
- `intersector.assume_curve_control_point_count(N)` **[NEW]**
- `geometry_type::curve` **[NEW]** enum value
- `intersection_type::curve` **[NEW]** enum value
- `MTLIndirectAccelerationStructureInstanceDescriptor.accelerationStructureID` **[NEW]** — assign AS object directly in GPU function

## Code Highlights

Creating a curve geometry descriptor:
```swift
let geometryDescriptor = MTLAccelerationStructureCurveGeometryDescriptor()
geometryDescriptor.controlPointBuffer = controlPointBuffer
geometryDescriptor.radiusBuffer = radiusBuffer
geometryDescriptor.indexBuffer = indexBuffer
geometryDescriptor.controlPointCount = controlPointCount
geometryDescriptor.segmentCount = segmentCount
geometryDescriptor.curveType = .round
geometryDescriptor.curveBasis = .bezier
geometryDescriptor.segmentControlPointCount = 4
```

Indirect instance acceleration structure descriptor (GPU-driven count):
```swift
var instanceASDesc = MTLIndirectInstanceAccelerationStructureDescriptor()
instanceASDesc.instanceDescriptorType = .indirect
instanceASDesc.maxInstanceCount = maxCount
instanceASDesc.instanceCountBuffer = instanceCountBuffer
instanceASDesc.instanceDescriptorBuffer = instanceDescriptorBuffer
```

Intersecting rays with curves (Metal Shading Language):
```metal
[[kernel]] void trace_rays(acceleration_structure<> as, ...) {
    intersector<curve_data> i;
    i.assume_geometry_type(geometry_type::curve | geometry_type::triangle);
    i.assume_curve_type(curve_type::round);
    i.assume_curve_basis(curve_basis::bezier);
    i.assume_curve_control_point_count(3);

    ray r(origin, direction);
    intersection_result<curve_data> result = i.intersect(r, as);
    if (result.type == intersection_type::curve) {
        float param = result.curve_parameter;
        // use param with basis function to get hit point
    }
}
```

Multi-level instance intersection:
```metal
[[kernel]] void trace_rays(acceleration_structure<instancing> as, ...) {
    intersector<instancing, max_levels<3>> i;
    intersection_result<instancing, max_levels<3>> result = i.intersect(r, as);
    for (uint l = 0; l < result.instance_count; ++l) {
        uint id = result.instance_id[l];
    }
}
```

## Takeaways
- Use curve geometry descriptors for hair, fur, and vegetation — they build faster, use less memory, and remain mathematically smooth at any camera distance compared to triangle approximations.
- Adopt multi-level instancing for hierarchical scene content (tree-of-leaves within a forest) to reduce instance counts by orders of magnitude and cut per-frame rebuild cost.
- Use the GPU-driven indirect instance descriptor for instance culling — eliminates the CPU round-trip to set the final instance count and unlocks fully GPU-driven acceleration structure builds.
- Batch acceleration structure builds on the same `MTLAccelerationStructureCommandEncoder` for parallelization, reuse scratch buffers between batches, and compact after the initial build to minimize memory footprint.

---
_Source: WWDC23 Session 10128 page (abstract, chapters, transcript, and code samples)._
