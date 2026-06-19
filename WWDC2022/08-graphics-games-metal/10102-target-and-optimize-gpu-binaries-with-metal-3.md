# Target and Optimize GPU Binaries with Metal 3
**WWDC22 · Session 10102** · [Watch](https://developer.apple.com/videos/play/wwdc2022/10102/)

_Platforms:_ iOS 16, iPadOS 16, macOS Ventura 13, tvOS 16

## Overview
This session introduces two Metal 3 improvements to GPU shader compilation: **offline binary generation** and the **optimize-for-size compiler option**. Together they address two recurring pain points in Metal-based apps and games: pipeline state object (PSO) creation stalls that cause stutters and long load screens, and unexpectedly large compile times for complex shaders.

Offline binary generation moves GPU binary creation entirely to Xcode project build time using a new JSON-based "pipelines script" artifact and the `metal` / `metal-tt` command-line tools. The resulting binary archive is shipped in the app bundle and loaded at runtime with zero just-in-time GPU compilation. The optimize-for-size option (`-Os` / `MTLLibraryOptimizationLevelSize`) limits code-expanding transformations (inlining, loop unrolling) to shrink large GPU programs and reduce compile times at the cost of potentially lower runtime performance — demonstrated with real gains in Blender's Cycles renderer.

## Key Topics

### GPU Binary Generation Stages
Metal compiles shaders through two stages: Metal source → Apple Intermediate Representation (AIR) `.air` file, then AIR → GPU binary. The first stage can be moved to build time by pre-compiling `.metal` to `.metallib`. The second (PSO creation) has historically been runtime-only; Metal 3 enables it at build time.

### Metal File System Cache and Binary Archives
Metal automatically caches GPU binaries in a file system cache. `MTLBinaryArchive` provides explicit control: add PSO descriptors to an archive, which generates and caches the binary. Archives can be serialized to disk and loaded at startup. Metal 3 extends this by generating archives entirely offline.

### Offline Compilation Workflow
1. Author or harvest a **Metal pipelines script** (`.mtlp-json`) — a JSON description of pipeline state descriptors equivalent to `MTLRenderPipelineDescriptor`, `MTLComputePipelineDescriptor`, etc.
2. Invoke `metal shaders.metal -N descriptors.mtlp-json -o archive.metallib` (from source) or `metal-tt shaders.metallib descriptors.mtlp-json -o archive.metallib` (from pre-compiled library).
3. Ship `archive.metallib` in the app bundle; load it at runtime via `MTLBinaryArchiveDescriptor.url`.

**Harvesting**: serialize archives during development using `MTLBinaryArchive.serialize(to:)`, then extract the JSON script with `metal-source -flatbuffers=json harvested.metallib -o descriptors.mtlp-json`.

### Forward Compatibility
Metal gracefully upgrades offline-generated binary archives asynchronously in the background during OS updates or app installs, ensuring binaries remain compatible with future GPU architectures.

### Optimize for Size (`-Os`)
New in Metal 3: `MTLLibraryOptimizationLevelSize` limits size-expanding compiler transformations (inlining, loop unrolling). Benefits for shaders with deep call paths and loops:
- Shorter compile times
- Smaller GPU binary → fewer instruction cache misses → potentially more threads in parallel
- Demonstrated on Blender Cycles: up to 1.4× faster setup time on Apple Silicon; up to 1.6× faster render times on Intel GPUs

Note: some shaders may see a runtime performance regression; measure both modes for each project.

## APIs & Frameworks

**Metal**

_Binary Archive_
- `MTLBinaryArchive` — existing protocol; archives GPU binaries
- `MTLBinaryArchiveDescriptor` — existing; `.url` property loads an offline-generated archive **[NEW use]**
- `MTLDevice.newBinaryArchive(descriptor:)` — existing; now loads offline archives via URL
- `MTLBinaryArchive.addRenderPipelineFunctions(descriptor:)` — existing; used in harvesting workflow
- `MTLBinaryArchive.serialize(to:)` — existing; used to produce harvestable archives

_Optimization Level_
- `MTLLibraryOptimizationLevel` **[NEW]** — enum
  - `.default` — existing aggressive performance optimization
  - `.size` **[NEW]** — optimize for smaller binary and shorter compile time
- `MTLCompileOptions.optimizationLevel: MTLLibraryOptimizationLevel` **[NEW]** — compile-time option

_Pipeline State (existing, used with offline archives)_
- `MTLRenderPipelineDescriptor`
- `MTLComputePipelineDescriptor`
- `MTLDevice.makeRenderPipelineState(descriptor:binaryArchives:)` — existing; loads GPU binary from archive without JIT compilation

**Command-Line Tools (Metal Toolchain)**
- `metal` — compiler; `-N <script>.mtlp-json` flag **[NEW]** for offline GPU binary generation
- `metal-tt` **[NEW]** — Metal translator tool; generates GPU binary from an existing `.metallib` + pipelines script
- `metal-source` — existing tool; `-flatbuffers=json` flag **[NEW]** extracts JSON pipelines script from a binary archive
- `-Os` flag **[NEW]** — optimize for size

**Pipelines Script Format**
- `.mtlp-json` — JSON file with `libraries.paths`, `pipelines.render_pipelines[]`, `pipelines.compute_pipelines[]` keys **[NEW]**

## Code Highlights

Harvesting a binary archive during development:
```objc
MTLBinaryArchiveDescriptor *archive_desc = [MTLBinaryArchiveDescriptor new];
id<MTLBinaryArchive> archive = [device newBinaryArchiveWithDescriptor:archive_desc error:&error];
[archive addRenderPipelineFunctionsWithDescriptor:pipeline_desc error:&error];
NSURL *url = [NSURL fileURLWithPath:@"harvested-binaryArchive.metallib"];
[archive serializeToURL:url error:&error];
```

Extracting a pipelines script from an archive:
```sh
metal-source -flatbuffers=json harvested-binaryArchive.metallib -o /tmp/descriptors.mtlp-json
```

Generating an offline GPU binary:
```sh
# From Metal source
metal shaders.metal -N descriptors.mtlp-json -o archive.metallib

# From a pre-compiled .metallib
metal-tt shaders.metallib descriptors.mtlp-json -o archive.metallib
```

Loading an offline-generated archive at runtime:
```objc
MTLBinaryArchiveDescriptor *desc = [MTLBinaryArchiveDescriptor new];
desc.url = [NSURL fileURLWithPath:@"archive.metallib"];
id<MTLBinaryArchive> binaryArchive = [device newBinaryArchiveWithDescriptor:desc error:&error];
```

Enabling optimize-for-size at runtime:
```objc
MTLCompileOptions *options = [MTLCompileOptions new];
options.optimizationLevel = MTLLibraryOptimizationLevelSize;
id<MTLLibrary> lib = [device newLibraryWithSource:source options:options error:&error];
```

## Takeaways
- Offline compilation moves all GPU binary generation to Xcode build time, eliminating PSO-creation stutters and dramatically shortening first launch and level-load times.
- Harvest a pipelines script from existing binary archives using `metal-source`, then regenerate offline with `metal` or `metal-tt` — no need to rewrite pipeline-creation code.
- Metal automatically forward-upgrades offline binary archives during OS updates, so no special re-submission is needed when GPU architectures change.
- Use `MTLLibraryOptimizationLevelSize` (or `-Os`) for shaders with deep call graphs or large loops; measure both modes, as runtime performance impact varies per shader.

---
_Source: WWDC22 Session 10102 page (abstract, chapter summaries, code samples, and resource links)._
