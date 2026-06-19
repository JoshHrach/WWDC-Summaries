# Build GPU Binaries with Metal
**WWDC20 · Session 10615** · [Watch](https://developer.apple.com/videos/play/wwdc2020/10615/)

_Platforms:_ iOS 14, iPadOS 14, macOS Big Sur 11, tvOS 14

## Overview
This session introduces two major additions to Metal's shader compilation model: Metal Binary Archives and Metal Dynamic Libraries. Both aim to drastically reduce Pipeline State Object (PSO) loading times — especially on first launch — and eliminate redundant compilation of shared utility code across multiple pipelines.

A real-world benchmark with Epic Games' Fortnite (11,000+ pipeline state objects) showed that pre-shipping a harvested binary archive reduced total PSO build time from 86 seconds to 3 seconds — a 28x speedup — on an Apple Silicon Mac mini.

The session also covers the expanded Metal Developer Tools toolchain (metal-libtool, metal-nm, metal-lipo) and announces Windows-hosted Metal Developer Tools, enabling cross-platform shader build pipelines.

## Key Topics

**Metal Binary Archives**
A new explicit PSO caching mechanism that stores GPU-compiled machine code for pipeline state objects. Archives can be created, populated, serialized to disk, harvested from a device, and deployed on compatible devices (same GPU + OS). Supported on Apple GPU Family 3 and Mac GPU Family 1.

Pipeline descriptors set the `binaryArchives` property before creation; the framework searches archives linearly for a cached binary. If not found, it falls back to runtime compilation and the system Metal Shader Cache (unless `failOnBinaryArchiveMiss` is set). Archives are memory-mapped, so they should be released when no longer needed to free virtual address space.

**Metal Dynamic Libraries**
A new `MTLDynamicLibrary` type enables GPU compute shader utility functions to be compiled once into machine code and dynamically linked across multiple compute pipelines. This eliminates duplicate compilation and duplicate machine code in memory for shared subroutines.

Dynamic libraries are serializable and deployable as app assets. At pipeline creation time, the Metal linker resolves imported function symbols against loaded dynamic libraries. `insertLibraries` on the compute pipeline descriptor enables function-level overriding (analogous to `DYLD_INSERT_LIBRARIES`).

**Metal Developer Tools Toolchain**
- `metal-libtool` — creates static libraries from AIR files (replaces `metal-ar`)
- `metal-nm` — inspects exported symbols in a MetalLib
- `metal-lipo` — creates/inspects fat universal binaries combining AIR and hardware-compiled slices
- Windows-hosted Metal Developer Tools **[NEW]** — enables Metal shader compilation on Windows infrastructure

**Symbol Visibility in Dynamic Libraries**
By default, all symbols are exported. Use `static`, anonymous namespaces, visibility attributes, or the linker's `exported-symbols-list` option to restrict exports and avoid runtime name collisions between libraries.

## APIs & Frameworks

### Metal **[NEW]**
- `MTLBinaryArchive` **[NEW]** — explicit PSO cache object
- `MTLBinaryArchiveDescriptor` **[NEW]** — descriptor for creating/loading an archive; `url` property (`nil` = new empty archive)
- `MTLDevice.makeBinaryArchive(descriptor:)` **[NEW]** — creates a `MTLBinaryArchive`
- `MTLBinaryArchive.addRenderPipelineFunctions(with:)` **[NEW]** — adds render PSOs to archive
- `MTLBinaryArchive.addComputePipelineFunctions(with:)` **[NEW]** — adds compute PSOs
- `MTLBinaryArchive.addTileRenderPipelineFunctions(with:)` **[NEW]** — adds tile render PSOs
- `MTLBinaryArchive.serialize(to:)` **[NEW]** — serializes archive to a URL on disk
- `MTLRenderPipelineDescriptor.binaryArchives` **[NEW]** — array of archives to search on PSO creation
- `MTLComputePipelineDescriptor.binaryArchives` **[NEW]** — same for compute pipelines
- `MTLPipelineOption.failOnBinaryArchiveMiss` **[NEW]** — returns nil if binary not found in archives (no fallback compilation)
- `MTLDynamicLibrary` **[NEW]** — GPU dynamic library object
- `MTLDevice.makeDynamicLibrary(library:)` **[NEW]** — compiles a `MTLLibrary` to a `MTLDynamicLibrary`
- `MTLDevice.makeDynamicLibrary(url:)` **[NEW]** — loads a serialized dynamic library from disk
- `MTLDynamicLibrary.serialize(to:)` **[NEW]** — saves dynamic library (AIR + GPU binary) to disk
- `MTLCompileOptions.libraryType` **[NEW]** — `.executable` (default) or `.dynamic`
- `MTLCompileOptions.installName` **[NEW]** — install name for dynamic library (`@executable_path/`, `@loader_path/`, or absolute path)
- `MTLCompileOptions.libraries` **[NEW]** — dynamic libraries to link at compile time (for symbol resolution checking)
- `MTLComputePipelineDescriptor.insertLibraries` **[NEW]** — libraries searched first for symbol resolution (function injection)
- `MTLDevice.supportsDynamicLibraries` **[NEW]** — feature query for dynamic library support

### Metal Developer Tools (command line)
- `metal` compiler — source → AIR (`.air`) → MetalLib (`.metallib`)
- `metal-libtool -static` **[NEW]** — creates a static AIR library archive
- Dynamic library linking: `metal -dynamiclib -install_name @executable_path/myLib.metallib`
- `metal-nm` **[NEW]** — lists exported symbols in a MetalLib
- `metal-lipo -info` **[NEW]** — shows architecture slices in a fat MetalLib
- `metal-lipo -create` **[NEW]** — combines multiple architecture slices into a universal MetalLib
- Metal Developer Tools for Windows **[NEW]** — Windows installer for offline shader compilation targeting Apple platforms

## Code Highlights

Creating and serializing a binary archive:
```swift
let desc = MTLBinaryArchiveDescriptor()
desc.url = nil  // new empty archive
let archive = try device.makeBinaryArchive(descriptor: desc)
try archive.addRenderPipelineFunctions(with: renderPipelineDescriptor)
let archiveURL = documentsURL.appendingPathComponent("binaryArchive.metallib")
try archive.serialize(to: archiveURL)
```

Reusing an archive on subsequent launches:
```swift
let desc = MTLBinaryArchiveDescriptor()
desc.url = archiveURL
let archive = try device.makeBinaryArchive(descriptor: desc)
renderPipelineDescriptor.binaryArchives = [archive]
let pipeline = try device.makeRenderPipelineState(descriptor: renderPipelineDescriptor)
```

Creating and using a dynamic library:
```swift
let options = MTLCompileOptions()
options.libraryType = .dynamic
options.installName = "@executable_path/myDynamicLibrary.metallib"
let utilityLib = try device.makeLibrary(source: dylibSrc, options: options)
let utilityDylib = try device.makeDynamicLibrary(library: utilityLib)

let kernelOptions = MTLCompileOptions()
kernelOptions.libraries = [utilityDylib]
let kernelLib = try device.makeLibrary(source: kernelStr, options: kernelOptions)
```

## Takeaways
- Metal Binary Archives provide explicit, developer-controlled PSO caching: pre-ship harvested GPU binaries to avoid all backend compilation on first launch, achieving up to 28x PSO build time reduction for large games.
- Metal Dynamic Libraries eliminate duplicate compilation of shared GPU utility code; a single dynamic library instance is linked by multiple compute pipelines at runtime, reducing both build time and memory usage.
- Use `metal-lipo` to combine AIR and hardware-specific slices into universal MetalLibs that fall back to runtime compilation on non-matching devices, making single-file deployment viable across device families.
- Metal Developer Tools are now available on Windows, enabling shader compilation workflows for Apple platforms in Windows-based build infrastructures.

---
_Source: WWDC20 Session 10615 page (abstract, chapter summaries, code samples, and resource links)._
