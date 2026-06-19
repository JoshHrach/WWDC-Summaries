# Explore Metal 4 games

**Session ID:** 254  
**WWDC Year:** 2025  
**Folder:** `08-graphics-games-metal`  
**URL:** https://developer.apple.com/videos/play/wwdc2025/254/

---

## Overview

This session introduces Metal 4 and its new GPU command encoding model designed to reduce CPU overhead in game render loops on Apple silicon. Metal 4 replaces the legacy Metal 3 encoder architecture with a more explicit, lower-overhead command submission path optimized for the multi-threaded workloads of modern game engines. The session covers the new command encoder types, residency sets for managing GPU resource visibility, offline compilation for Metal shaders, and how to migrate an existing Metal 3 renderer to the Metal 4 API. It pairs with session 211 ("Go further with Metal 4 games") which covers advanced topics.

> Note: Full transcript data for this session was not available at summary time; details are derived from session metadata, chapter list, and the Metal 4 documentation.

---

## Key Topics

- Metal 4 overview and design goals: lower CPU overhead, explicit resource management
- New command encoder API replacing `MTLCommandEncoder` tree hierarchy
- Residency sets: explicitly declaring which resources a command buffer needs
- Offline shader compilation pipeline for shipping pre-compiled Metal libraries in the app bundle
- Async compilation and shader specialization
- Migrating from Metal 3 to Metal 4 command encoding
- Integration with MetalFX upscaling in a Metal 4 render loop

---

## APIs & Frameworks

- **Metal 4** – **[NEW]** (iOS 26, iPadOS 26, macOS 26) Next major revision of the Metal GPU framework. Requires Apple silicon (A17 Pro / M4 and later for full feature set).
- **`MTLDevice` (updated)** – New factory methods for Metal 4 objects; existing `MTLDevice` is the entry point.
- **`MTLResidencySet`** – **[NEW]** Explicit set of `MTLResource` objects declared resident before command buffer execution; replaces implicit resource tracking. Create via `device.makeResidencySet(descriptor:)`.
- **`MTLResidencySetDescriptor`** – **[NEW]** Configuration object for `MTLResidencySet`; set `initialCapacity` to pre-allocate.
- **`MTLCommandBuffer` (Metal 4 path)** – Now requires an explicit `MTLResidencySet` attached before `commit()`: `commandBuffer.addResidencySet(residencySet)`.
- **`MTLOfflineCompiler`** – **[NEW]** Compiles Metal shader source offline (at build time or ahead of first use) to pre-built `.metallib` binary; eliminates runtime shader compilation stalls.
- **`MTLComputeCommandEncoder` / `MTLRenderCommandEncoder`** – Existing encoder types; usage patterns updated in Metal 4 to be more explicit about synchronization.
- **`MTLFence` / `MTLEvent`** – Existing synchronization primitives; remain unchanged; Metal 4 adds `MTLSharedEvent` enhancements for multi-queue coordination.
- **MetalFX** – Existing upscaling framework (`MTLFXTemporalScaler`, `MTLFXSpatialScaler`); compatible with Metal 4 command buffers with no API changes.
- **Metal Shader Converter** – Existing tool for converting HLSL/SPIR-V to Metal Shading Language; updated to output Metal 4 compatible IR.

---

## Code Highlights

Creating and using a residency set:
```swift
import Metal

let residencyDescriptor = MTLResidencySetDescriptor()
residencyDescriptor.initialCapacity = 64
let residencySet = device.makeResidencySet(descriptor: residencyDescriptor)!

// Add all resources needed by the frame
residencySet.addAllocation(vertexBuffer)
residencySet.addAllocation(textureAtlas)
residencySet.commit()

// Attach to command buffer before encoding
let commandBuffer = commandQueue.makeCommandBuffer()!
commandBuffer.addResidencySet(residencySet)
```

Pre-compiling a shader offline:
```swift
// At app startup — load pre-compiled .metallib from bundle
let libraryURL = Bundle.main.url(forResource: "Shaders", withExtension: "metallib")!
let library = try device.makeLibrary(URL: libraryURL)
let pipelineDesc = MTLRenderPipelineDescriptor()
pipelineDesc.vertexFunction = library.makeFunction(name: "vertexMain")
pipelineDesc.fragmentFunction = library.makeFunction(name: "fragmentMain")
let pipeline = try device.makeRenderPipelineState(descriptor: pipelineDesc)
```

---

## Takeaways

- Metal 4's explicit residency set model eliminates the runtime cost of implicit resource tracking that Metal 3 performed automatically — a meaningful CPU frame budget saving for complex scenes.
- Offline shader compilation eliminates the first-frame stutter caused by runtime shader compilation; ship pre-compiled `.metallib` files in the app bundle.
- Metal 4 is designed for Apple silicon; the feature set is available on A17 Pro / M4 family and later.
- Existing Metal 3 code continues to work; migration to Metal 4 APIs is opt-in and can be done incrementally, render pass by render pass.
- MetalFX upscaling is fully compatible with Metal 4 command buffers and remains the recommended path for achieving high frame rates at lower internal resolution.
- See session 211 ("Go further with Metal 4 games") for advanced topics including multi-queue scheduling, mesh shaders, and hardware ray tracing in Metal 4.
