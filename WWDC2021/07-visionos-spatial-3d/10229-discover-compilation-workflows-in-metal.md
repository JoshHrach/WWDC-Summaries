# Discover compilation workflows in Metal
**WWDC21 · Session 10229** · [Watch](https://developer.apple.com/videos/play/wwdc2021/10229/)

_Platforms:_ iOS 15, iPadOS 15, macOS Monterey 12

## Overview
Metal's shader compilation model has grown to match GPU workload complexity. This session introduces five compilation workflow improvements in Metal for iOS 15 and macOS Monterey: dynamic libraries extended to render and tile pipelines, function pointers extended to render and tile pipelines on Apple Silicon, binary archives that now persist function pointer binaries to disk, private linked functions for compile-time-optimized static linking, and the new function stitching API for generating shader functions from computation graphs at runtime without Metal-source string manipulation.

## Key Topics

### Dynamic Libraries for Render Pipelines **[NEW]**
- Dynamic libraries (`.metallib`) allow utility functions to be compiled once and shared across multiple render, tile, and compute pipelines without recompilation.
- Previously limited to compute pipelines (WWDC20); now available for render and tile pipelines in iOS 15 / macOS Monterey on Apple GPU family 6+.
- Functions in a dynamic library are declared `extern` in the consuming shader and resolved at pipeline creation time.
- Can be swapped at runtime by changing which libraries are in the pipeline descriptor's `preloadedLibraries` array.
- Create: `newDynamicLibrary(url:)` from AIR; serialize to disk with `serializeToURL(_:)`; reload with `newDynamicLibrary(url:)`.

### Function Pointers for Render Pipelines **[NEW]**
- Function pointers let a shader call functions unknown at compile time (only the signature is needed), enabling fully dynamic dispatch and runtime extensibility.
- Previously compute-only (WWDC20); extended to render and tile pipeline stages in iOS 15 / macOS Monterey on Apple GPU family 6+.
- Functions can be provided as AIR (compiler can optimize with static linking) or as GPU binaries (fast pipeline creation, pre-compiled).
- Workflow: declare `MTLFunctionDescriptor` → set `options = .compileToBinary` → create function from library → add to `fragmentLinkedFunctions.binaryFunctions` → create `MTLVisibleFunctionTable` → bind handles → bind table in command encoder → call via `visible_function_table` in shader.
- `supportAddingFragmentBinaryFunctions = YES` enables **incremental pipeline extension**: create a new pipeline from an existing one that adds additional binary functions without full recompilation.
- When using deep call chains with binary functions, set `maxCallStackDepth` explicitly to avoid stack overflows and achieve optimal resource use.

### Binary Archives for Function Pointers **[NEW]**
- `MTLBinaryArchive` now supports storing compiled visible and intersection function binaries, not just pipelines.
- `addFunction(with:from:)` serializes a function to the archive.
- On subsequent launches, pass the archive in `functionDescriptor.binaryArchives`; `newFunction(with:)` returns the cached binary immediately.
- `failOnBinaryArchiveMiss` option: set to `true` to return `nil` rather than recompile on a cache miss (useful for CI / shipping builds that should always hit the cache).
- Reduces memory cost and eliminates runtime compilation overhead for function pointers.

### Private Linked Functions **[NEW]**
- Statically linked visible functions that are private to a pipeline and cannot be placed in a `MTLVisibleFunctionTable`.
- Unlike `linkedFunctions.functions` (which requires function pointer support on the GPU), private functions require no function pointer hardware and are available on **all devices** in iOS 15 / macOS Monterey.
- Because the function is private and cannot be used as a function pointer, the backend compiler can perform full inlining and optimization, yielding better runtime performance.
- Add via `linkedFunctions.privateFunctions` on the pipeline descriptor.

### Function Stitching **[NEW]**
- Generates new Metal functions directly in AIR from a Directed Acyclic Graph (DAG) of precompiled **stitchable functions** at runtime—no Metal source string generation or Metal-to-AIR frontend compilation at runtime.
- Functions marked `[[stitchable]]` in Metal source are compiled to AIR at build time; the stitching runtime combines them into a new function.
- Two graph node types: `MTLFunctionStitchingInputNode` (function arguments), `MTLFunctionStitchingFunctionNode` (function calls with data/control edges).
- Generates a `MTLLibrary` from a `MTLStitchedLibraryDescriptor` containing the stitchable functions and the graph; the resulting `MTLFunction` can itself be stitchable for further composition.
- Available on **all devices** in iOS 15 / macOS Monterey.
- Ideal for apps that currently generate Metal source strings dynamically (e.g., material systems, procedural effects) — replace string manipulation with a structured graph API.

## APIs & Frameworks

**Metal**
- `MTLDevice.newDynamicLibrary(_:)` / `newDynamicLibrary(url:)` — create dynamic library from AIR or file **[extended to render/tile]**
- `MTLDynamicLibrary.serializeToURL(_:)` — persist dynamic library to disk **[extended to render/tile]**
- `MTLRenderPipelineDescriptor.fragmentPreloadedLibraries` / `.vertexPreloadedLibraries` — dynamic library list per stage **[NEW for render]**
- `MTLFunctionDescriptor.options = .compileToBinary` — compile function to GPU binary **[extended to render/tile]**
- `MTLRenderPipelineDescriptor.fragmentLinkedFunctions.binaryFunctions` — list of binary functions callable from fragment stage **[NEW]**
- `MTLRenderPipelineDescriptor.supportAddingFragmentBinaryFunctions` — enable incremental pipeline extension **[NEW]**
- `MTLRenderPipelineDescriptor.maxCallStackDepth` — specify max function pointer call depth **[NEW]**
- `MTLRenderPipelineState.newVisibleFunctionTable(descriptor:stage:)` — create visible function table **[extended to render]**
- `MTLRenderPipelineState.functionHandle(function:stage:)` — create function handle **[extended to render]**
- `MTLVisibleFunctionTable.setFunction(_:at:)` — insert function handle **[existing]**
- `MTLRenderCommandEncoder.setFragmentVisibleFunctionTable(_:atBufferIndex:)` — bind table in encoder **[NEW]**
- `MTLRenderPipelineState.newRenderPipelineState(additionalBinaryFunctions:)` — incremental pipeline creation **[NEW]**
- `MTLBinaryArchive.addFunction(with:from:)` — add function pointer binary to archive **[NEW]**
- `MTLFunctionDescriptor.binaryArchives` — archives to search before compiling **[NEW]**
- `MTLFunctionOptions.failOnBinaryArchiveMiss` — fail rather than recompile on cache miss **[NEW]**
- `MTLRenderPipelineDescriptor.fragmentLinkedFunctions.privateFunctions` — statically linked private functions **[NEW]**
- `MTLFunctionStitchingInputNode` — input node in a stitching graph **[NEW]**
- `MTLFunctionStitchingFunctionNode` — function call node in a stitching graph **[NEW]**
- `MTLFunctionStitchingGraph` — DAG of stitchable functions **[NEW]**
- `MTLStitchedLibraryDescriptor` — descriptor for creating a stitched library **[NEW]**
- `MTLDevice.newLibrary(descriptor:)` — create stitched library from descriptor **[NEW]**

**Metal Shading Language**
- `[[stitchable]]` function attribute — marks a visible function as usable in function stitching graphs **[NEW]**
- `visible_function_table<Signature>` buffer type — shader-side function pointer table **[extended to render/tile]**

## Code Highlights

Declaring external functions and calling them in a fragment shader (dynamic library pattern):
```metal
extern float4 foo(FragmentInput input);
extern float4 bar(FragmentInput input);

fragment float4 main(FragmentInput input [[stage_in]]) {
    switch (condition(input)) {
        case 0: return foo(input);
        case 1: return bar(input);
    }
}
```

Incremental pipeline creation to add a binary function without full recompilation:
```objc
renderPipeDesc.supportAddingFragmentBinaryFunctions = YES;

MTLRenderPipelineFunctionsDescriptor *extraDesc = [MTLRenderPipelineFunctionsDescriptor new];
extraDesc.fragmentAdditionalBinaryFunctions = @[bat];

id<MTLRenderPipelineState> renderPipeline2 =
    [renderPipeline1 newRenderPipelineStateWithAdditionalBinaryFunctions:extraDesc error:&error];
```

Function stitching — building a computation graph and generating a new AIR function at runtime:
```objc
// Input nodes
inputs[0] = [[MTLFunctionStitchingInputNode alloc] initWithArgumentIndex:0];

// Function call nodes
n2 = [[MTLFunctionStitchingFunctionNode alloc]
    initWithName:@"FunctionC" arguments:@[n0, n1] controlDependencies:@[]];

// Graph
graph = [[MTLFunctionStitchingGraph alloc]
    initWithFunctionName:@"ResultFunction" nodes:@[n0, n1] outputNode:n2 attributes:@[]];

// Generate library directly in AIR
MTLStitchedLibraryDescriptor *desc = [MTLStitchedLibraryDescriptor new];
desc.functions = @[stitchableFunctions];
desc.functionGraphs = @[graph];

id<MTLLibrary> lib = [device newLibraryWithDescriptor:desc error:&error];
id<MTLFunction> stitchedFunction = [lib newFunctionWithName:@"ResultFunction"];
```

## Takeaways
- Dynamic libraries are the right tool for stable utility code shared across pipelines; function pointers are for pipelines that need to call unknown functions at runtime.
- Private functions give the best runtime performance for static linking without requiring function pointer hardware—use them on all GPU families.
- Binary archives now cache function pointer binaries to disk; combine with `failOnBinaryArchiveMiss` in shipping builds to guarantee zero runtime compilation.
- Function stitching replaces Metal source string generation: precompile snippets to AIR at build time, compose them into new functions at runtime via a structured graph API, and skip the Metal frontend entirely.

---
_Source: WWDC21 Session 10229 page (abstract, full transcript, code samples, and resource links)._
