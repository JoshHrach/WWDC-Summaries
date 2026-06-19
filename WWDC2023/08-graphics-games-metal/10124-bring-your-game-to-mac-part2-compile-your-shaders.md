# Bring Your Game to Mac, Part 2: Compile Your Shaders
**WWDC23 · Session 10124** · [Watch](https://developer.apple.com/videos/play/wwdc2023/10124/)

_Platforms:_ macOS Sonoma 14, macOS Ventura 13.3

## Overview
Part 2 of the three-part "Bring Your Game to Mac" series focuses on converting HLSL shaders to Metal and finalizing GPU binaries ahead of time to eliminate in-game compilation stalls. The session introduces the **Metal Shader Converter** — a new tool that converts DXIL (DirectX Intermediate Language) directly to Metal IR, supporting all shader stages and enabling a complete end-to-end shader pipeline from HLSL to distributable Metal libraries.

The second half covers the **GPU Binary Compiler** toolchain (`metal-tt`), which compiles Metal libraries into device-specific GPU binaries at game build time, eliminating load-time compilation and reducing in-game hitches from pipeline state object (PSO) creation.

## Key Topics

### Metal Shader Converter **[NEW]**
- Converts DXIL to Metal IR at the binary level — fast, and avoids generating Metal IR at game runtime.
- Used alongside the open-source **DXC** (DirectX Compiler) tool: compile HLSL to DXIL with DXC, then convert DXIL to a Metal library with Metal Shader Converter.
- Generated Metal libraries are identical to those produced by the Metal compiler — they integrate natively with all Metal APIs.
- Available as: (1) CLI tool for shell scripts and one-off conversion; (2) dynamic library (C interface) for integration into custom asset build pipelines, on both macOS and Windows.

### Shader Stage Support
- Traditional pipeline: vertex, hull/domain (tessellation), geometry shaders → mapped to **Mesh Shaders** in Metal
- Modern pipeline: amplification, mesh shaders, compute shaders
- Ray tracing stages: ray generation, intersection, closest-hit, miss shaders
- Geometry and tessellation shaders are emulated via Metal Mesh Shaders (object + mesh stages), including the traditionally fixed-function tessellator. Metal now supports linking visible functions to object and mesh stages.

### Resource Binding / Argument Buffers
- HLSL register declarations and root signatures are mapped to Metal Argument Buffers.
- Two layout modes:
  - **Automatic** — shader converter lays out resources sequentially in one argument buffer; simplest to adopt.
  - **Explicit** — matches a Direct3D root signature layout; needed for separate resource tables or bindless resources. Supports embedded raw buffers and 32-bit constants.
- Argument buffer is a CPU/GPU shared resource — use a **bump allocator** (large Metal buffer, sub-allocated per frame, shadowed per frame-in-flight) to avoid race conditions.
- Use `useResources` (plural) or `useHeap` instead of individual `useResource` calls for efficient residency flagging.

### Mixing Metal Libraries
- Metal Shader Converter libraries and MSL-compiled libraries can be mixed in the same app and same pipeline, enabling incremental adoption and use of MSL-only features like programmable blending and tile shading.

### Finalizing GPU Binaries Ahead of Time
- PSO creation at runtime = on-device compilation = load-time stalls and in-game hitches.
- **`metal-tt`** — GPU Binary Compiler tool that finalizes Metal libraries into device-specific GPU binary archives at build time.
- Workflow: Metal library + **pipeline script** (JSON configuration matching the Metal PSO descriptor) → `metal-tt` → Metal binary archive with GPU binaries for all target devices.
- Binary archives are associated with `MTLRenderPipelineDescriptor.binaryArchives` / `MTLComputePipelineDescriptor.binaryArchives`.
- Metal automatically selects the correct GPU binary for the player's device from the archive.
- **Fallback**: if a binary is missing from the archive, Metal falls back to on-device compilation automatically (no crash).
- `MTLPipelineOption.failOnBinaryArchiveMiss` — test mode: returns `nil` PSO instead of falling back, for verifying archive completeness.
- **Pipeline script harvesting**: use `metal-source` to extract pipeline scripts from binary archives recorded while running the game on device.
- **OS compatibility**: generate a binary archive per major OS version; load the appropriate one at runtime using availability checks. Metal upgrades archives in the background after OS updates.
- Toolchain (`metal`, `metal-shaderconverter`, `metal-tt`, `metal-source`) all available on both macOS and Windows.

## APIs & Frameworks

### Metal Shader Converter **[NEW]**
- `metal-shaderconverter` CLI tool — DXIL → Metal library conversion
- Metal Shader Converter dynamic library — C interface for integration into build pipelines (macOS and Windows)
- DXC (DirectX Compiler) — open source; compiles HLSL to DXIL; used upstream of shader converter
- Runtime companion header — C header for shader converter runtime features
- Supports: vertex, hull, domain, geometry, amplification, mesh, compute, ray generation, intersection, closest-hit, miss shader stages

### Metal IR and Compiler Toolchain
- `metal` compiler — compiles MSL to Metal IR / Metal library (existing)
- Metal library (`.metallib`) — Metal IR package; works with converted and MSL-compiled shaders interchangeably
- Metal IR — Metal's intermediate representation; platform-agnostic; compiled to GPU binaries separately
- `metal-tt` — GPU Binary Compiler; finalizes Metal libraries into device-specific GPU binaries in Metal binary archives
- `metal-source` — extracts pipeline scripts from recorded binary archives
- Metal binary archive (`MTLBinaryArchive`) — contains device-specific GPU binaries

### Metal API
- `MTLDevice.makeLibrary(URL:options:)` — load Metal library from disk
- `MTLRenderPipelineDescriptor.binaryArchives` — attach binary archives for PSO creation
- `MTLComputePipelineDescriptor.binaryArchives` — attach binary archives
- `MTLComputePipelineDescriptor` — compute PSO descriptor
- `MTLDevice.newComputePipelineState(descriptor:options:error:)` — create compute PSO
- `MTLPipelineOption.failOnBinaryArchiveMiss` **[NEW]** — returns nil if GPU binary not found in archive
- `MTLSamplerDescriptor.supportsArgumentBuffers` — required when using samplers in argument buffers
- `useResources(_:usage:)` — flag residency of multiple resources efficiently
- `useHeap(_:usage:)` — flag residency of entire heap
- `MTLVisibleFunctionTable` — used with mesh shader visible functions **[enhanced]**
- `MTLMeshRenderPipelineDescriptor` — mesh shader pipeline descriptor

### MetalKit
- `MTKTextureLoader` — can load files as texture arrays (helpful for HLSL texture-as-array emulation)

## Code Highlights

Testing for binary archive hit:
```objc
MTLComputePipelineDescriptor *computeDesc = [MTLComputePipelineDescriptor new];
computeDesc.binaryArchives = @[existingBinaryArchive];
computeDesc.computeFunction = computeFn;
id<MTLComputePipelineState> computePS =
    [device newComputePipelineStateWithDescriptor:computeDesc
                    options:MTLPipelineOptionFailOnBinaryArchiveMiss
                    error:&err];
if (computePS == nil) {
    // Binary archive is missing compiled shader
}
```

Loading the correct binary archive per OS version:
```objc
if (@available(macOS 14, *)) {
    computeDesc.binaryArchives = @[binaryArchive_macOS14];
} else {
    computeDesc.binaryArchives = @[binaryArchive_macOS13_3];
}
```

## Takeaways
- Metal Shader Converter converts DXIL to Metal IR at the binary level — the same Metal library format used by MSL-compiled shaders, enabling seamless integration.
- Geometry and tessellation shaders are emulated via Metal Mesh Shaders, with shader converter doing the translation automatically.
- Use Argument Buffers in automatic or explicit layout mode to map HLSL register/root-signature resource binding to Metal.
- Pre-compile GPU binaries with `metal-tt` + pipeline scripts at build time to eliminate load-time compilation stalls and in-game PSO hitches.

---
_Source: WWDC23 Session 10124 page (abstract, chapter summaries, code samples, and resource links)._
