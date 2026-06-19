# Go bindless with Metal 3
**WWDC22 · Session 10101** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10101/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
The bindless rendering model aggregates all scene resources (textures, buffers, acceleration structures) into argument buffers and heaps, then lets shaders freely navigate them at runtime via GPU virtual addresses and resource IDs. This unlocks advanced techniques like ray tracing and reduces CPU overhead from per-draw-call resource binding.

Metal 3 introduces four major improvements to the bindless workflow: direct argument buffer writing without encoder objects, acceleration structure heap allocation, enhanced shader validation layer diagnostics for missing residency, and new dependency viewer tools in Xcode 14's Metal Debugger. The session also covers two performance optimizations — unretained command buffer resources and untracked heap resources with manual synchronization — that reduce CPU and GPU overhead for long-lived aggregate resources.

## Key Topics

**Simplified argument buffer writing (Metal 3)** — Previously required an `MTLArgumentEncoder` object. Now, write directly into a buffer using C structs: store 64-bit GPU virtual addresses (`buffer.gpuAddress`) for buffer pointers and `MTLResourceID` values for textures. Works on any device with Argument Buffers Tier 2 (Mac 2016+, A13 Bionic+). Unbounded arrays work the same way — allocate `sizeof(Struct) * count` bytes, cast `contents` to the struct pointer type, and fill via a loop.

**Acceleration structures from heaps (Metal 3 new)** — `MTLAccelerationStructure` can now be allocated from `MTLHeap` alongside buffers and textures. Use `heapAccelerationStructureSizeAndAlignWithDescriptor:` to get the required size/alignment. All structures in the heap can then be made resident with a single `useHeap:` call instead of individual `useResource:` calls — significant CPU savings on the render thread.

**Shader validation layer residency errors (Metal 3 new)** — When an indirectly accessed resource is not resident at GPU execution time, the validation layer now produces an error message naming the shader function, pass, Metal source file/line, buffer label, buffer size, and the fact it was not resident. With the debug breakpoint enabled, Xcode shows the exact shader line where the access failed.

**Unretained resources** — Command buffers normally retain all referenced resources, preventing deallocation while the GPU works. For long-lived resources (e.g., entire-level geometry) that the app already manages, this is unnecessary overhead. Create command buffers with `commandBufferWithUnretainedReferences` to skip this. CPU overhead measured at ~2% reduction in command buffer lifecycle time.

**Untracked resources and manual synchronization** — Tracked heaps conservatively serialize all passes that touch any resource within the heap ("false sharing"), reducing GPU parallelism. Mark heap resources as `.untracked` (the default for heaps) and use fine-grained synchronization primitives to express only the actual dependencies: `MTLFence` (single-queue, split barrier), `MTLEvent` (multi-queue, same device), `MTLSharedEvent` (multi-device + CPU), or `MTLMemoryBarrier` (within a single pass). Never use a fragment-stage barrier on Apple GPUs (use a Fence instead).

**Metal Debugger — new Dependency Viewer (Xcode 14)** — Graphical view of the frame workload showing each pass as a node and resource dependencies as edges. Solid lines = data flow; dotted lines = synchronization dependencies introducing GPU waits. Clicking a heap highlights false-sharing between passes that have no real dependency. Filtering to "Synchronization only" view makes false sharing immediately visible.

**Metal Debugger — Resource Access List** — For any draw call, switch to "Accessed" mode to see only the resources that shader actually accessed from argument buffers, with their access type. Paired with the Shader Debugger (line-by-line execution trace), this can identify wrong argument buffer indexing.

## APIs & Frameworks

### Metal

**Argument buffers (simplified in Metal 3)**
- `MTLBuffer.gpuAddress` **[NEW accessible without encoder]** — 64-bit GPU virtual address; store as `uint64_t` in CPU-side struct
- `MTLResource.gpuResourceID` **[NEW accessible without encoder]** — `MTLResourceID` value for textures; store directly in struct
- `MTLDevice.argumentBuffersSupport` — query for `.tier2` to confirm direct-write support
- `MTLBuffer.contents` — raw pointer to cast to struct type for direct writing
- `__METAL_VERSION__` macro — conditional compilation in shared CPU/GPU header files

**Acceleration structures in heaps**
- `MTLDevice.heapAccelerationStructureSizeAndAlignWithDescriptor:` **[NEW]** — returns `MTLSizeAndAlign` for heap allocation
- `MTLHeap.newAccelerationStructureWithSize:` — allocate an acceleration structure from a heap
- `MTLComputeCommandEncoder.useHeap:stages:` — make all heap resources resident in one call

**Command buffer resource management**
- `MTLCommandQueue.commandBufferWithUnretainedReferences` **[NEW]** — command buffer that does not retain referenced resources; caller guarantees resource lifetimes
- `MTLCommandBufferDescriptor.retainedReferences` — `false` to create unretained command buffer

**Hazard tracking**
- `MTLResourceHazardTrackingMode.untracked` — opt resource/heap out of automatic hazard tracking; manual synchronization required
- `MTLHeap` — hazard tracking defaults to `.untracked` (most efficient)
- `MTLFence` — split barrier for single-queue producer/consumer synchronization
- `MTLEvent` — synchronize across multiple queues on the same device
- `MTLSharedEvent` — synchronize across multiple devices and between GPU and CPU
- `MTLRenderCommandEncoder.memoryBarrierWithScope:after:before:` — within-pass synchronization; avoid after fragment stage on Apple GPUs

**Metal Debugger (Xcode 14)**
- Dependency Viewer **[NEW]** — graph view of frame workload; data flow (solid edges) vs. synchronization (dotted edges); filters by dependency type
- Resource Access List **[NEW]** — "Accessed" mode on draw calls shows only resources accessed by the shader
- Shader Debugger — line-by-line shader execution trace; shows resource accesses per line

**Metal Shading Language**
- `constant T* ptr` — GPU-side pointer type corresponding to CPU `uint64_t` address
- Unbounded array parameter `constant Mesh* meshes [[buffer(n)]]` — freely indexable at any length
- Scene aggregation struct: `constant Mesh* meshes; constant Material* materials;` — single buffer binding for entire scene

## Code Highlights

Writing an argument buffer directly (no encoder needed in Metal 3):
```objc
// CPU-side struct (uint64_t for buffer ptrs, MTLResourceID for textures)
struct Mesh { uint64_t normals; };

NSUInteger meshArgumentSize = sizeof(struct Mesh);
id<MTLBuffer> meshArgumentBuffer = [device newBufferWithLength:meshArgumentSize
                                                        options:storageMode];
struct Mesh* meshes = (struct Mesh *)(meshArgumentBuffer.contents);
meshes->normals = normalBuffer.gpuAddress + normalBufferOffset;
```

Shared CPU/GPU struct with conditional compilation:
```objc
#if __METAL_VERSION__
#define CONSTANT_PTR(x) constant x*
#else
#define CONSTANT_PTR(x) uint64_t
#endif
struct Mesh { CONSTANT_PTR(packed_float3) normals; };
```

Heap allocation query for acceleration structures:
```objc
MTLSizeAndAlign sizeAndAlign =
    [device heapAccelerationStructureSizeAndAlignWithDescriptor:descriptor];
```

## Takeaways
- Metal 3 eliminates the `MTLArgumentEncoder` requirement; write argument buffers directly as C structs using `gpuAddress` for buffers and `gpuResourceID` for textures on Argument Buffers Tier 2 devices (Mac 2016+, A13+).
- Allocate `MTLAccelerationStructure` objects from a `MTLHeap` to make all of them resident with a single `useHeap:` call, cutting CPU overhead significantly.
- Use `commandBufferWithUnretainedReferences` for long-lived scene resources to eliminate unnecessary ARC overhead (~2% command buffer CPU time).
- Opt heaps to `.untracked` and use `MTLFence`/`MTLEvent` to express only real dependencies; this removes false sharing and allows the GPU to overlap passes that touch different resources within the same heap.

---
_Source: WWDC22 Session 10101 page (abstract, transcript, code samples, and resource links)._
